# Catalog

The Catalog resource serves **instance-wide reference data** — the value lists a client needs to
fill a dropdown or to validate its input before sending a write request. It is the one resource
that exposes no user data at all: everything it returns is the same for every token on the
instance.

- **Base path:** `/api/v2/catalog`
- **Scope:** none — any valid token may read it
- **Since Version:** <span class="wl-since wl-since-dev">dev, not released yet</span>

!!! note
    Read the [API v2 overview](index.md) first for authentication, the response envelope and
    error codes.

!!! warning "Wavelog is not the master of this data"
    The `dxcc` and `subdivisions` topics hand out publicly available reference data — the DXCC
    entity list and the ADIF subdivision list. Wavelog only mirrors it, as a convenience for a
    client that is talking to the API anyway and would otherwise need a second data source just
    to fill a dropdown.

    On top of that Wavelog **enriches** the mirrored lists with information of its own — zones,
    coordinates and similar additions the source list does not carry — so a response is not a
    field-for-field copy of any official list either.

    What you get is therefore a copy, and it is only as fresh as the instance's last country
    file update — every instance updates on its own schedule, so two instances may disagree.
    Whenever your application needs data it can rely on — award checking, publications,
    anything a user acts on — please take it from the official sources (the ARRL DXCC list and
    the ADIF specification) instead of from a Wavelog instance.

## No scope required

Like the [Token](token.md) (whoami) resource, Catalog needs a **valid token but no particular
scope**. There is no `catalog:read` to grant — the scope does not exist and never appears in the
token dialog. A token minted with nothing but `qso:read` can still read the contest catalog.

The reasoning: a scope protects *your* data, and there is none here. The contest catalog is the
same list the Contesting module shows to every user of the instance, so gating it would only force
every client to request a scope it could never be refused.

The resource is **read-only**. `POST`, `PATCH` and `DELETE` return `405 method_not_allowed` with
`Allow: GET`.

## Topics

The `?topic=` parameter selects a list. Called **without** a topic, the endpoint enumerates the
ones it has, so a client can discover them instead of hard-coding names:

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/catalog" \
     -H "Authorization: Bearer wl2_your_token_here"
```

```json
{
  "data": { "topics": ["contest", "dxcc", "subdivisions"] }
}
```

An unknown topic returns `400 validation_error` with the valid names in `error.details.allowed`.

| Topic | Returns | Parameters | Since |
| --- | --- | --- | --- |
| `contest` | The active contest catalog | — | <span class="wl-since wl-since-dev">dev</span> |
| `dxcc` | The currently valid DXCC entities | `order=name\|prefix` (optional) | <span class="wl-since wl-since-dev">dev</span> |
| `subdivisions` | The primary administrative subdivisions of one DXCC entity | `dxcc=<entity number>` (required) | <span class="wl-since wl-since-dev">dev</span> |

## Topic: `contest`

The contests the Contesting module offers, i.e. exactly the values the [Contest](contest.md)
resource accepts when creating or updating a session:

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/catalog?topic=contest" \
     -H "Authorization: Bearer wl2_your_token_here"
```

```json
{
  "data": [
    {
      "id": 1,
      "contest": "DARC-WAG",
      "name": "Worked All Germany Contest"
    },
    {
      "id": 57,
      "contest": "CQ-WW-SSB",
      "name": "CQ Worldwide DX Contest, SSB"
    }
  ],
  "meta": {
    "topic": "contest"
  }
}
```

| Field | Meaning |
| --- | --- |
| `contest` | **ADIF contest name** — maintained as *ADIF Name* in *Contesting* → *Contest Administration*, and the value to send as `contest` in a [Contest](contest.md) write request |
| `name` | Human-readable name, for display |
| `id` | Catalog id; **instance-local**, see the warning below |

The `contest` value is what the ADIF specification calls `CONTEST_ID` — not to be confused with
this API's numeric `contest_id`. The [Contest](contest.md#the-contest-name) page maps the three
names against each other.

!!! warning "Use `contest`, not `id`, to address a contest"
    The catalog ids are assigned per instance — id `42` is a different contest on another Wavelog.
    A client that syncs between instances must key on the ADIF name. The id is included only
    because the [Contest](contest.md) resource accepts `contest_id` as an alternative for
    single-instance use.

Only **active** contests are listed. An inactive one is rejected by the Contest resource on write
anyway, so it would be a value a client could see but never use. A session that still references a
deactivated contest keeps reporting its name in the session object's `contest` / `contest_name`
fields, so nothing becomes unreadable.

The order follows the Contesting module's own: the generic catch-all entry first, then
alphabetically by name.

## Topic: `dxcc`

The currently valid DXCC entities — deleted ones and the `0` entity are not listed. Use it to fill
an entity dropdown, e.g. in a station-profile form:

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/catalog?topic=dxcc" \
     -H "Authorization: Bearer wl2_your_token_here"
```

```json
{
  "data": [
    {
      "adif": 287,
      "name": "SWITZERLAND",
      "prefix": "HB",
      "continent": "EU",
      "cqz": 14,
      "ituz": 28,
      "lat": 47,
      "long": 7.5
    }
  ],
  "meta": {
    "topic": "dxcc"
  }
}
```

| Field | Meaning |
| --- | --- |
| `adif` | **ADIF entity number** — the value the [Station](station.md) resource expects as `dxcc` |
| `name` | Entity name as maintained by the DXCC module |
| `prefix` | Main prefix of the entity |
| `continent` | Continent abbreviation (`EU`, `NA`, `AS`, ...) |
| `cqz` / `ituz` | Default CQ and ITU zone of the entity |
| `lat` / `long` | Reference coordinates, decimal degrees; `long` is positive east |

The default order is by `name`. Pass `?order=prefix` to sort by main prefix instead; any other
value returns `400 validation_error` with the accepted ones in `error.details.allowed`.

`cqz`, `ituz`, `lat` and `long` are a *reference* for the whole entity, not for a specific
station. A station in a large entity can legitimately sit in a different zone — for callsign-based
resolution use the [Lookup](lookup.md) resource, which evaluates the actual prefix.

!!! note "Use the official sources where you can"
    DXCC data is public and Wavelog serves it here purely for convenience, so a client does not
    have to ship its own copy. Nothing guarantees this instance runs the latest DXCC update.

## Topic: `subdivisions`

The primary administrative subdivisions (ADIF `STATE`) of **one** DXCC entity, selected with the
required `dxcc` parameter:

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/catalog?topic=subdivisions&dxcc=287" \
     -H "Authorization: Bearer wl2_your_token_here"
```

```json
{
  "data": [
    { "state": "AG", "name": "Aargau" },
    { "state": "AI", "name": "Appenzell Innerrhoden" },
    { "state": "AR", "name": "Appenzell Ausserrhoden" }
  ],
  "meta": {
    "topic": "subdivisions"
  }
}
```

| Field | Meaning |
| --- | --- |
| `state` | **ADIF subdivision code** — the value the [QSO](qso.md) resource expects in `state` |
| `name` | Human-readable subdivision name, for display |

A missing or non-numeric `dxcc` returns `400 validation_error`. An entity that simply has no
subdivisions returns an empty list, not an error — most entities have none. Pair this topic with
the `dxcc` one if you also need the entity's name or zones.

Only subdivisions that are **currently valid** are listed. Deprecated ones still exist in Wavelog
so an old QSO keeps showing the state it was logged with, but they are not valid choices for
anything a client creates, so the catalog leaves them out.

## Adding topics

Topics are registered in one map inside `Catalog_resource::index()`. Further lists — bands, modes,
propagation modes — can be added there without a new route and without a new scope: a client that
already polls `/api/v2/catalog` discovers them through the topic list.
