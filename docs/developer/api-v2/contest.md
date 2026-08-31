# Contest

The Contest resource manages **contest sessions** — the Contesting module's grouping of QSOs into
a contest run — and the links between a session and its QSOs. It is the API counterpart to the
*Contesting* pages in the web UI and lets external tools (loggers, offline mirrors, sync clients)
read and replicate contest sessions instead of rebuilding them by hand via *Import Historical
Contests*.

- **Base path:** `/api/v2/contest`
- **Scopes:** `contest:read`, `contest:write`, `contest:delete`
- **Since version:** <span class="wl-since wl-since-dev">dev, not released yet</span>

!!! note
    Read the [API v2 overview](index.md) first for authentication, the response envelope and
    error codes.

## Endpoints

| Verb | Path | Scope | Purpose | Since version |
| --- | --- | --- | --- | --- |
| `GET` | `/api/v2/contest` | `contest:read` | List the token owner's contest sessions | <span class="wl-since wl-since-dev">dev</span> |
| `GET` | `/api/v2/contest/{id}` | `contest:read` | A single session, including its linked QSO ids | <span class="wl-since wl-since-dev">dev</span> |
| `POST` | `/api/v2/contest` | `contest:write` | Create a session, optionally linking QSOs | <span class="wl-since wl-since-dev">dev</span> |
| `PATCH` | `/api/v2/contest/{id}` | `contest:write` | Partial update; link and unlink QSOs | <span class="wl-since wl-since-dev">dev</span> |
| `DELETE` | `/api/v2/contest/{id}` | `contest:delete` | Delete a session | <span class="wl-since wl-since-dev">dev</span> |

A session belonging to another user returns `404 not_found`, never `403` — the API never confirms
that a foreign id exists.

## The session object

```json
{
  "id": 17,
  "contest": "DARC-WAG",
  "contest_name": "Worked All Germany Contest",
  "time_start": "2026-10-17 15:00:00",
  "time_end": "2026-10-18 15:00:00",
  "station_id": 3,
  "comment": "SO ABHP",
  "settings": {
    "exchangetype": "Serialexchange",
    "copyexchangeto": "dok",
    "exchangefields": ["serial", "exchange"],
    "callbook_lookup": true,
    "custom_name": "",
    "serial_per_band": false,
    "serial_scope": "station"
  },
  "qso_count": 412,
  "created_at": "2026-10-17 14:52:11",
  "updated_at": "2026-10-18 15:04:37"
}
```

| Field | Meaning |
| --- | --- |
| `id` | Session id, stable within this Wavelog instance |
| `contest` | **ADIF contest name**, e.g. `DARC-WAG` — the value used to address a contest, see [The contest name](#the-contest-name) |
| `contest_name` | Human-readable contest name from the catalog |
| `time_start` / `time_end` | Session window, `YYYY-MM-DD HH:MM:SS` |
| `station_id` | Station location the session belongs to |
| `comment` | Free-text note |
| `settings` | Session settings, see [below](#session-settings) |
| `qso_count` | Number of QSOs currently linked to the session |
| `created_at` / `updated_at` | Timestamps maintained by Wavelog |

!!! tip "Address contests by ADIF name, not by id"
    The numeric contest catalog ids are **instance-local** — id `42` is a different contest on
    another Wavelog. Write requests accept `contest_id` as an alternative, but a client that
    syncs between instances should always use `contest`. Responses carry both the ADIF name and
    the resolved catalog name.

    The available contests are served by
    [`GET /api/v2/catalog?topic=contest`](catalog.md#topic-contest) — no scope required, so any
    valid token can fill a contest dropdown.

### The contest name

One and the same value goes by three different names, which is easy to trip over:

| Where | Called |
| --- | --- |
| This API | `contest` |
| Web UI, *Contesting* → *Contest Administration* | *ADIF Name* (*Contest ADIF Name* in the edit form) |
| Exported ADIF record | `CONTEST_ID` |

So a contest an administrator created with *ADIF Name* = `DARC-WAG` is addressed as
`"contest": "DARC-WAG"` here, and every QSO exported from such a session carries
`<CONTEST_ID:8>DARC-WAG`.

!!! warning "`CONTEST_ID` is not `contest_id`"
    `CONTEST_ID` is the **ADIF field name** for that text value. This API's `contest_id` is
    something else entirely: the **numeric** catalog id. Sending `{"contest_id": "DARC-WAG"}`
    fails with `400 validation_error`, and so does `{"contest": 57}`.

### Session settings

`settings` is a JSON object. On read it is always returned **complete**, merged over the module
defaults, so a client never has to know which keys were stored. On write only the keys you send
are validated and merged over the existing values.

| Key | Default | Values |
| --- | --- | --- |
| `exchangefields` | `["serial"]` | Non-empty array of `serial`, `gridsquare`, `exchange` |
| `exchangetype` | `Serial` | **Derived**, see below |
| `copyexchangeto` | `""` | `""`, `dok`, `locator`, `qth`, `name`, `age`, `state`, `power` |
| `callbook_lookup` | `true` | Boolean |
| `custom_name` | `""` | String |
| `serial_per_band` | `false` | Boolean |
| `serial_scope` | `"station"` | `station` or `operator` |

`exchangefields` is an array and therefore the one place where the API's usual
[scalar-only body rule](index.md#request-bodies) does not apply — `settings`, `qso_ids`,
`link_qso_ids` and `unlink_qso_ids` may be nested, every other field must stay scalar.

`exchangetype` is a **derived legacy field**. Whatever you send is ignored and recomputed from
`exchangefields`, exactly as the web UI does it, so the two can never drift apart:

| `exchangefields` | Resulting `exchangetype` |
| --- | --- |
| `["serial"]` | `Serial` |
| `["exchange"]` | `Exchange` |
| `["serial", "exchange"]` | `Serialexchange` |
| `["serial", "gridsquare"]` | `Serialgridsquare` |
| `["exchange", "gridsquare"]` | `Exchangegridsquare` |
| `["serial", "gridsquare", "exchange"]` | `SerialGridExchange` |

An unknown settings key or an invalid value returns `400 validation_error` with the individual
messages in `error.details.errors`.

## List sessions

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/contest" \
     -H "Authorization: Bearer wl2_your_token_here"
```

Sessions are returned newest first (by `time_start`, then `id`). The list is **not paginated** and
carries no `qso_ids` — fetch a single session for that.

| Parameter | Default | Meaning |
| --- | --- | --- |
| `station_id` | all owned by the token user | Comma-separated station ids; ownership-checked |
| `since_id` | `0` | Only sessions with `id` greater than this — incremental sync |

A `station_id` the token does not own is rejected with `403 forbidden` rather than silently
dropped.

## Fetch a single session

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/contest/17" \
     -H "Authorization: Bearer wl2_your_token_here"
```

The response is the session object plus `qso_ids`, the sorted list of linked QSO primary keys —
the same ids the [QSO](qso.md) resource uses, so a client can mirror the linkage:

```json
{
  "data": {
    "id": 17,
    "contest": "DARC-WAG",
    "qso_count": 3,
    "qso_ids": [48190, 48200, 48215]
  }
}
```

## Create a session

```bash
curl -X POST "https://<WAVELOG_URL>/index.php/api/v2/contest" \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{
           "contest": "DARC-WAG",
           "time_start": "2026-10-17 15:00",
           "time_end": "2026-10-18 15:00",
           "station_id": 3,
           "comment": "SO ABHP",
           "settings": {
             "exchangefields": ["serial", "exchange"],
             "copyexchangeto": "dok"
           }
         }'
```

| Field | Required | Notes |
| --- | --- | --- |
| `contest` **or** `contest_id` | yes | ADIF contest name (preferred) or catalog id |
| `time_start` | yes | `YYYY-MM-DD HH:MM` or `YYYY-MM-DD HH:MM:SS` |
| `time_end` | yes | same format |
| `station_id` | yes | Must belong to the token owner |
| `comment` | no | Defaults to empty |
| `settings` | no | Merged over the defaults |
| `qso_ids` | no | QSOs to link right away, see [Linking QSOs](#linking-and-unlinking-qsos) |

On success the answer is `201 Created` with a `Location` header pointing at the new session and
the created session in `data`.

The contest must exist **and be active** in the contest catalog; an unknown or deactivated contest
returns `400 validation_error`. The [Catalog](catalog.md#topic-contest) resource lists the
accepted values. The `qso_ids` list is validated *before* the session is created,
so a rejected link list can never leave a stale empty session behind.

## Update a session

```bash
curl -X PATCH "https://<WAVELOG_URL>/index.php/api/v2/contest/17" \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{"time_end": "2026-10-18 16:00", "settings": {"serial_per_band": true}}'
```

Only the fields present in the body change. `settings` is merged key by key over the stored
settings — sending `{"serial_per_band": true}` leaves every other setting untouched. A body with
no editable field and no link list returns `400 validation_error`.

## Linking and unlinking QSOs

The URL space has no sub-paths, so the linkage travels in the body of a `PATCH`:

```bash
curl -X PATCH "https://<WAVELOG_URL>/index.php/api/v2/contest/17" \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{"link_qso_ids": [48190, 48200], "unlink_qso_ids": [47001]}'
```

The response is the session object with the outcome appended — the extra fields only appear when
the request actually contained a link list:

```json
{
  "data": {
    "id": 17,
    "qso_count": 413,
    "linked": 2,
    "skipped": [],
    "unlinked": 1
  }
}
```

| Field | Meaning |
| --- | --- |
| `linked` | Number of QSOs newly linked to this session |
| `skipped` | Ids that were **already linked to a different session** and were left alone |
| `unlinked` | Number of links removed |

Linking is **idempotent**: an id already linked to *this* session is a silent no-op, so a sync
client can re-send its full list without special-casing. An id linked to *another* session is
never stolen — it lands in `skipped`.

Linking also sets the QSO's `CONTEST_ID` field to the session's contest, and unlinking clears it
again (only on QSOs that were really linked to this session), mirroring the attach/detach workflow
of the [advanced logbook](../../user-guide/logbook/advanced-logbook.md).

QSO ids that do not belong to the token owner return `403 forbidden` with the offending ids in
`error.details.qso_ids`; the whole list is validated before anything is applied, so a rejected
request changes nothing.

## Delete a session

```bash
curl -X DELETE "https://<WAVELOG_URL>/index.php/api/v2/contest/17" \
     -H "Authorization: Bearer wl2_your_token_here"
```

Returns `204 No Content`. By default the linked QSOs **stay in the logbook** and only lose their
session link — the same as unchecking the box in the web UI.

To tear the whole run down, including the QSOs:

```bash
curl -X DELETE "https://<WAVELOG_URL>/index.php/api/v2/contest/17?delete_qsos=true" \
     -H "Authorization: Bearer wl2_your_token_here"
```

!!! warning "`delete_qsos=true` also needs `qso:delete`"
    Removing logbook contacts is a QSO-resource power. A token holding only `contest:delete` gets
    `403 insufficient_scope` with `details.required_scope = "qso:delete"`, and nothing is deleted.
    Deleted QSOs are gone — there is no undo.

## Clubstation tokens

For a [clubstation](clubstation.md) token the member's permission level applies on top of the
scopes, matching what the web UI allows:

| Operation | Required level |
| --- | --- |
| List and read sessions | any member |
| Create, update or delete a session | **9 (Club Officer)** |
| Link and unlink QSOs | any member — but only **their own** QSOs |

A member below officer level that tries to edit the session itself gets
`403 insufficient_club_permission`. Sessions are shared club infrastructure, so their fields stay
officer-only, while attaching one's own QSOs to a running session does not — the same split the
advanced logbook uses.

## Errors

| Status | Code | When |
| --- | --- | --- |
| 400 | `validation_error` | Missing or malformed field, unknown/inactive contest, invalid settings, empty update |
| 403 | `forbidden` | `station_id` or a QSO id not accessible for this token |
| 403 | `insufficient_scope` | Missing `contest:*` scope, or `qso:delete` for `delete_qsos=true` |
| 403 | `insufficient_club_permission` | Clubstation member below officer level editing a session |
| 404 | `not_found` | No such session for this token owner |
