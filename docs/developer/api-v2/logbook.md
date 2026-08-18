# Logbook

The Logbook resource manages your **logbooks** — the containers that group
[station locations](station.md) together. A logbook decides *which* locations are shown as one
log: the dashboard, the logbook views, the awards and the statistics all follow the logbook that
is currently active.

- **Base path:** `/api/v2/logbook`
- **Scopes:** `logbook:read`, `logbook:write`, `logbook:delete`
- **Since version:** <span class="wl-since wl-since-dev">dev, not released yet</span>

All operations are scoped to the token owner. A logbook belonging to another user is treated as
*not found*.

!!! note
    Read the [API v2 overview](index.md) first for authentication, the response envelope and
    error codes.

## Logbooks and station locations

The two are easy to mix up, so it is worth being explicit:

- A [**station location**](station.md) is what a QSO is logged against — callsign, grid, DXCC.
  QSOs belong to it.
- A **logbook** owns no QSOs at all. It is a named set of station locations, and it exists so you
  can view several of them (home, portable, contest call, …) as one log — or keep them apart.

Deleting a logbook therefore never touches a QSO; deleting a station location does.

## Endpoints

| Verb | Path | Scope | Purpose | Since version |
| --- | --- | --- | --- | --- |
| `GET` | `/api/v2/logbook` | `logbook:read` | List all your logbooks | <span class="wl-since wl-since-dev">dev</span> |
| `GET` | `/api/v2/logbook/{id}` | `logbook:read` | Fetch a single logbook | <span class="wl-since wl-since-dev">dev</span> |
| `POST` | `/api/v2/logbook` | `logbook:write` | Create a logbook | <span class="wl-since wl-since-dev">dev</span> |
| `PATCH` | `/api/v2/logbook/{id}` | `logbook:write` | Rename, set active, change linked locations | <span class="wl-since wl-since-dev">dev</span> |
| `DELETE` | `/api/v2/logbook/{id}` | `logbook:delete` | Delete a logbook | <span class="wl-since wl-since-dev">dev</span> |

List endpoints are not paginated — users typically have only a handful of logbooks.

## The logbook object

```json
{
  "id": 3,
  "name": "Contest",
  "active": true,
  "station_ids": [1, 4]
}
```

| Field | Notes |
| --- | --- |
| `id` | Logbook id |
| `name` | Logbook name |
| `active` | Read-only: `true` for the owner's currently active logbook |
| `station_ids` | The [station locations](station.md) this logbook contains |

`active` is read-only and has a **write counterpart of its own**, `set_active`. The two are
deliberately different keys, so a `GET` result can be sent straight back through `PATCH` without
silently reassigning your active logbook.

## List logbooks

`GET /api/v2/logbook`

```bash
curl https://<WAVELOG_URL>/index.php/api/v2/logbook \
     -H "Authorization: Bearer wl2_your_token_here"
```

```json
{
  "data": [
    { "id": 1, "name": "Home", "active": false, "station_ids": [1] },
    { "id": 3, "name": "Contest", "active": true, "station_ids": [1, 4] }
  ]
}
```

A single logbook is available at `GET /api/v2/logbook/{id}` and returns the same object.

## Create a logbook

`POST /api/v2/logbook`

**Required field:** `name`. `station_ids` is optional — leave it out to create an empty logbook
and link the locations later.

```bash
curl -X POST https://<WAVELOG_URL>/index.php/api/v2/logbook \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{ "name": "Portable", "station_ids": [4] }'
```

- Your **first** logbook automatically becomes the active one.
- On success the API returns `201 Created`, a `Location` header and the created object in `data`.
- A name you already use returns `409 conflict`.
- A `station_ids` entry that is not one of your station locations returns `403 forbidden`, with
  the offending id in `details.station_ids`. Foreign ids are rejected, never silently dropped.

## Update a logbook

`PATCH /api/v2/logbook/{id}` — partial update. Send at least one of the three writable fields;
a body with none of them returns `400 validation_error`.

| Field | Notes |
| --- | --- |
| `name` | New name; a duplicate returns `409 conflict` |
| `station_ids` | **Replaces** the linked locations — see below |
| `set_active` | `true` makes this the owner's active logbook |

```bash
curl -X PATCH https://<WAVELOG_URL>/index.php/api/v2/logbook/3 \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{ "name": "Contest 2026", "set_active": true }'
```

The updated logbook object is returned in `data`.

!!! warning "`station_ids` replaces the whole list"
    It is not a "add these locations" call: the list you send becomes the list that is stored.
    Sending `[]` unlinks every location, and adding one location means sending the ids the
    logbook already has plus the new one. Take `station_ids` from a `GET`, change it, send it
    back.

Everything is validated before the first write, so a rejected `PATCH` never leaves the logbook
half-updated — a bad `station_ids` entry cannot land a rename you did not get to keep.

## Delete a logbook

`DELETE /api/v2/logbook/{id}`

```bash
curl -X DELETE https://<WAVELOG_URL>/index.php/api/v2/logbook/3 \
     -H "Authorization: Bearer wl2_your_token_here"
```

- The **active** logbook cannot be deleted and returns `409 conflict`. Activate another one
  first, using `set_active` on a `PATCH`.
- On success the API returns `204 No Content`.
- **QSOs are not touched.** They belong to the station locations, which keep existing — only the
  grouping is gone.

## Clubstation tokens

For a [clubstation](clubstation.md) token every verb of this resource — **reads included** —
requires permission level **9 (Club Officer)**:

| Operation | Level 3 / 6 | Level 9 |
| --- | --- | --- |
| `GET /api/v2/logbook` | `403` | ✅ |
| `GET /api/v2/logbook/{id}` | `403` | ✅ |
| `POST` / `PATCH` / `DELETE` | `403` | ✅ |

This mirrors the web UI, where the whole logbook management screen is officer-only: a member
below that level manages no club logbooks there and manages none through the API either. The
refusal is `403 insufficient_club_permission`, carrying `details.required_level`.

The `logbook:*` scopes are not even offered when a token is created from a clubstation session
below officer level — a token that could only ever answer `403` is not worth minting.

## Errors

| Status | Code | When |
| --- | --- | --- |
| 400 | `validation_error` | Missing `name`, a non-numeric `station_ids` entry, or a `PATCH` with no writable field |
| 403 | `forbidden` | `station_ids` contains a station location not accessible for this token |
| 403 | `insufficient_scope` | Missing `logbook:read` / `logbook:write` / `logbook:delete` |
| 403 | `insufficient_club_permission` | Clubstation member below officer level |
| 404 | `not_found` | No such logbook for this token owner |
| 409 | `conflict` | Duplicate logbook name, or deleting the active logbook |
