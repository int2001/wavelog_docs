# Club

The Club resource manages the **members of a clubstation** and the permission
level each of them holds. Reading the list is the v2 equivalent of the v1
`list_clubmembers` endpoint; the write operations are the API counterpart of the
club permission page in the web UI.

- **Base path:** `/api/v2/club`
- **Scopes:** `club:read`, `club:write`, `club:delete`

!!! note
    Read the [API v2 overview](index.md) first for authentication, the response
    envelope and error codes.

!!! warning "Only on instances with clubstations enabled"
    The whole resource only exists when the administrator has switched on the
    clubstation feature (`special_callsign`). Without it every path below
    answers `404 not_found` — exactly like an unknown resource — and the
    `club:*` scopes are not offered when creating a token.

## Endpoints

The path id addresses the **member**, using their `user_id`.

| Verb | Path | Scope | Purpose |
| --- | --- | --- | --- |
| `GET` | `/api/v2/club` | `club:read` | List the clubstation's members — or, for an administrator who named no club, [the clubstations](#which-clubstations-are-there) |
| `GET` | `/api/v2/club/{user_id}` | `club:read` | Fetch a single member |
| `POST` | `/api/v2/club` | `club:write` | Add a member to the clubstation |
| `PATCH` | `/api/v2/club/{user_id}` | `club:write` | Change a member's permission level |
| `DELETE` | `/api/v2/club/{user_id}` | `club:delete` | Remove a member from the clubstation |

!!! note "There is no `PUT`"
    As on [Station](station.md) and [QSO](qso.md), the API has no full replace.
    A membership has exactly one changeable field, so `PATCH` is all it needs.

## Who may call this

The same two roles that can open the club permission page in the web UI:

| Token | Which clubstation | `?club_id=` |
| --- | --- | --- |
| **Club token** whose creator is an **officer** (level 9) | the token owner | optional, and only that same club |
| **Personal token** of a **Wavelog administrator** | whichever `club_id` names | **mandatory** |

A club token belongs to the clubstation (owner) but was created by a member
acting on its behalf (creator), so the club is implicit there. An administrator
manages every clubstation and has no implicit one — they always name it.

Anything else is refused with `403 forbidden`:

```json
{
  "error": {
    "code": "forbidden",
    "message": "Token is not a club officer"
  }
}
```

That covers a personal token of a regular user, a club token below officer level
and a token whose membership has since been revoked alike. See
[Clubstations](clubstation.md) for the permission levels.

!!! note "The scopes are only offered where they are usable"
    A session that matches neither role is not shown the `club:*` checkboxes
    when creating a token — a regular user outside a clubstation, or a member
    below officer level inside one, has no club they could address.

## Selecting the clubstation

`?club_id=` works the same way on every verb, including `POST`, `PATCH` and
`DELETE` — it selects the club, so it stays out of the request body.

```bash
# Administrator: name the club
curl "https://<WAVELOG_URL>/index.php/api/v2/club?club_id=5" \
     -H "Authorization: Bearer wl2_admin_token"

# Officer: the club token already knows
curl "https://<WAVELOG_URL>/index.php/api/v2/club" \
     -H "Authorization: Bearer wl2_club_token"
```

| Situation | Response |
| --- | --- |
| Administrator without `club_id`, on `GET /api/v2/club` | the [clubstation list](#which-clubstations-are-there) |
| Administrator without `club_id`, anywhere else | `400 validation_error` |
| `club_id` present but not numeric | `400 validation_error` |
| `club_id` is not a clubstation, or does not exist | `404 not_found` |
| Club token, `club_id` names a different club | `403 forbidden` |
| Club token, `club_id` names its own club | accepted |

## Which clubstations are there

`GET /api/v2/club` **without** `club_id`, as an administrator: the directory of
clubstations to pick a `club_id` from. Every other call needs one, so refusing
this with a `400` would leave nowhere to look it up.

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/club" \
     -H "Authorization: Bearer wl2_admin_token"
```

```json
{
  "data": [
    { "club_id": 5, "callsign": "N0CLUB", "member_count": 4 },
    { "club_id": 8, "callsign": "N0DX", "member_count": 0 }
  ],
  "meta": { "count": 2 }
}
```

`member_count` counts members of any permission level. A club token never sees
this list — it has exactly one club, so `GET /api/v2/club` returns that club's
members.

!!! warning
    The response contains every member's **email address** alongside their name
    and locator, and `club:write` lets a token hand out officer rights. Neither
    scope should be given to a third-party tool lightly.

## The member object

```json
{
  "user_id": 42,
  "user_firstname": "Alex",
  "user_lastname": "Example",
  "user_locator": "JN47",
  "callsign": "N0CALL",
  "user_name": "alex",
  "user_email": "alex@example.org",
  "permission_level": 9,
  "user_language": "english"
}
```

| Field | Notes |
| --- | --- |
| `user_id` | The member's user id — the path id of this resource |
| `user_firstname` / `user_lastname` | Member's real name |
| `user_locator` | Maidenhead locator |
| `callsign` | The member's own callsign, not the clubstation's |
| `user_name` | Login/username |
| `user_email` | Member's email address |
| `permission_level` | Clubstation permission level: `3`, `6` or `9` |
| `user_language` | Member's UI language |

## List the members

`GET /api/v2/club` with a club selected — implicitly by a club token, or
explicitly with `?club_id=` as an administrator. Without one, an administrator
gets [the clubstation list](#which-clubstations-are-there) instead.

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/club" \
     -H "Authorization: Bearer wl2_club_token"
```

```json
{
  "data": [
    {
      "user_id": 42,
      "user_firstname": "Alex",
      "user_lastname": "Example",
      "user_locator": "JN47",
      "callsign": "N0CALL",
      "user_name": "alex",
      "user_email": "alex@example.org",
      "permission_level": 9,
      "user_language": "english"
    }
  ],
  "meta": { "count": 1 }
}
```

`GET /api/v2/club/{user_id}` returns a single one of these objects, or
`404 not_found` when that user is not a member of the clubstation.

## Add a member

`POST /api/v2/club`

**Required fields:** `user_id`, `permission_level`.

```bash
curl -X POST https://<WAVELOG_URL>/index.php/api/v2/club \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{ "user_id": 42, "permission_level": 6, "notify": true }'
```

- On success the API returns `201 Created`, a `Location` header and the created
  member object in `data`.
- A user who is **already** a member returns `409 conflict` — use `PATCH` to
  change their level.
- `permission_level` must be `3`, `6` or `9`; anything else returns
  `400 validation_error` with the accepted values in `details.allowed`.

### Writable fields

| Field | Notes |
| --- | --- |
| `user_id` | The user to add (`POST` only) |
| `permission_level` | `3` = Club Member, `6` = Club Member ADIF, `9` = Club Officer |
| `notify` | Optional, see [Notifying the member](#notifying-the-member) |

## Change a permission level

`PATCH /api/v2/club/{user_id}`

```bash
curl -X PATCH https://<WAVELOG_URL>/index.php/api/v2/club/42 \
     -H "Authorization: Bearer wl2_your_token_here" \
     -H "Content-Type: application/json" \
     -d '{ "permission_level": 9 }'
```

The updated member object is returned in `data`. Unlike the web UI this never
creates a membership: a user who is not a member yet returns `404 not_found`.
Adding one is `POST`.

## Remove a member

`DELETE /api/v2/club/{user_id}`

```bash
curl -X DELETE https://<WAVELOG_URL>/index.php/api/v2/club/42 \
     -H "Authorization: Bearer wl2_your_token_here"
```

- On success the API returns `204 No Content`.
- A user who is not a member returns `404 not_found`.

!!! warning
    Removing a member also deletes the **API keys, API tokens and rig control
    sessions** they created for this clubstation, exactly like the web UI. A
    token deleted this way is gone for good: re-adding the member restores their
    permission, but they have to create a new token. See
    [Clubstations](clubstation.md#losing-membership).

    Their personal account, their personal tokens and their own QSOs are
    untouched — but the QSOs they logged under the club callsign stay with the
    club.

## Notifying the member

`POST` and `PATCH` accept an optional `notify` flag. When it is `true`, Wavelog
sends the member the same email the web UI does — "added to the club" on
`POST`, "permission changed" on `PATCH`, in the member's own language.

The outcome is reported in the response meta:

```json
{
  "data": { "user_id": 42, "permission_level": 9, "…": "…" },
  "meta": { "notified": true }
}
```

A failed email does **not** fail the request: the permission was granted either
way, and `meta.notified` is `false`. The field is omitted entirely when `notify`
was not requested.

!!! note
    This needs working email settings on the instance. Without them
    `meta.notified` is always `false`.

## Guard rails

Beyond the access check, the API refuses a few operations the web UI would
allow:

| Situation | Officer | Administrator |
| --- | --- | --- |
| Acting on **your own** membership | `403 forbidden` | allowed |
| The clubstation's own `user_id` as the target | `400 validation_error` | `400 validation_error` |
| The target user is itself a clubstation | `400 validation_error` | `400 validation_error` |
| The target user does not exist | `404 not_found` | `404 not_found` |
| The club is managed by an identity provider | `409 conflict` on every write | `409 conflict` on every write |

!!! note "Why an officer cannot change their own membership"
    Demoting or removing themselves over the API would lock the club out of its
    own permission management, with only a Wavelog administrator able to undo
    it. The web UI still allows it, where the consequence is visible — and an
    administrator reaches every club regardless, so the rule would only get in
    their way.

!!! note "SSO-managed clubs"
    When memberships come from an identity provider
    (`auth_header_clubstation_direct`), they are granted and revoked on login.
    A write through the API would be silently undone, so it is refused with
    `409 conflict` — the same reason the web UI disables the form.

## Wavelog administrators

An administrator (`user_type` 99) manages every clubstation, exactly as in the
web UI. Two ways to get there:

- **A personal token plus `?club_id=`** — the usual way. Create the token from
  your normal session; the `club:*` scopes are offered because you are an
  administrator. Every request then names the club.
- **A club token** created while switched into a clubstation. The club is
  implicit, and it works even without an entry on that club's permission page:
  the API waves administrators through the membership check, just like the web
  UI does.

!!! note
    An administrator who *does* hold an explicit permission level on a club is
    treated by that level, not as an administrator — a level below 9 is refused.
    Setting yourself to Club Member is a deliberate choice, and the API respects
    it. Use a personal token with `?club_id=` if you want the administrator path
    instead.
