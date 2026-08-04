# Confirmation

The Confirmation resource is a **read-only** endpoint that lists the QSL confirmations the token
owner has received — one row per **(QSO, confirmation type)** pair. It is the API counterpart to
the web UI's *Confirmations* page: a QSO confirmed via both LoTW and eQSL appears as **two rows**
here, exactly as it does in that table.

- **Base path:** `/api/v2/confirmation`
- **Scope:** `confirmation:read`

!!! note
    Read the [API v2 overview](index.md) first for authentication, the response envelope and
    error codes.

## What this endpoint is (and is not)

It is easy to confuse with two other features, so it is worth being explicit:

- [Statistic `?profile=confirmations`](statistic.md#confirmations) returns **aggregate counts** per
  type (how many QSOs confirmed by LoTW, by eQSL, …). This endpoint returns the **per-QSO records**
  themselves — callsign, dates, mode/band and the matching type — so a client can render the same
  view a human sees in the browser.
- The [QSO list's](qso.md) `?qsl_filter=` parameter narrows the QSO list to "QSOs that have at
  least one of these confirmations", but it does not surface **which** type matched, nor **when**
  the confirmation arrived. This endpoint does.

## Endpoint

| Verb | Path | Scope | Purpose |
| --- | --- | --- | --- |
| `GET` | `/api/v2/confirmation` | `confirmation:read` | List QSL confirmations, newest first |

This is a **list-only** endpoint. There is no single-item URL — a request like
`/api/v2/confirmation/42` returns `404 not_found`, because a confirmation has no stable id of its
own (it is identified by the QSO plus its type).

## Filters

All parameters are optional and can be combined:

| Parameter | Default | Values |
| --- | --- | --- |
| `type` | all | Comma-separated list of `lotw`, `eqsl`, `qsl`, `qrz`, `clublog` |
| `station_id` | all owned by the token user | Comma-separated station ids; ownership-checked |
| `since` | none | `YYYY-MM-DD` — only confirmations **received** on or after this date |
| `qso_since` | none | `YYYY-MM-DD` — only QSOs **made** on or after this date |
| `qso_until` | none | `YYYY-MM-DD` — only QSOs **made** on or before this date |
| `band` | all | A band such as `20m`, or `SAT` for satellite QSOs |
| `mode` | all | A mode or submode, e.g. `CW` or `FT8` |
| `callsign` | none | Exact match on the worked callsign |
| `page` | `1` | Page number |
| `per_page` | `100` | Page size, hard-capped at `1000` |

The `?type=` values are the lowercase keys (`lotw`, `eqsl`, …). The `type` field **in the
response** carries the human-readable label instead (`LoTW`, `eQSL`, `QSL Card`, `QRZ.com`,
`Clublog`) — the same words the web UI uses.

`since` and `qso_since` answer different questions, just as they do on the [Statistic
confirmations topic](statistic.md#confirmations). `since` filters on the date a confirmation
**arrived** — "which confirmations came in this week" — applied per type against that type's own
received-date column. `qso_since` / `qso_until` filter on the date of the **QSO** itself. Both
ends are inclusive and can be used on their own.

An invalid `type` value or a malformed date returns `400 validation_error`. Note that HRDLog is
**not** a confirmation type: Wavelog only uploads to it, there is no received status to list.

## Example

```bash
curl "https://<WAVELOG_URL>/index.php/api/v2/confirmation?type=lotw,eqsl&since=2026-07-01" \
     -H "Authorization: Bearer wl2_your_token_here"
```

```json
{
  "data": [
    {
      "qso_id": 48215,
      "callsign": "DL1ABC",
      "qso_date": "2026-07-28 18:30:00",
      "mode": "CW",
      "band": "20m",
      "gridsquare": "JN49",
      "confirmation_date": "2026-08-02",
      "type": "LoTW"
    },
    {
      "qso_id": 48200,
      "callsign": "OE3XYZ",
      "qso_date": "2026-07-26 19:05:00",
      "mode": "SSB",
      "band": "40m",
      "confirmation_date": "2026-08-01",
      "type": "eQSL"
    },
    {
      "qso_id": 48200,
      "callsign": "OE3XYZ",
      "qso_date": "2026-07-26 19:05:00",
      "mode": "SSB",
      "band": "40m",
      "confirmation_date": "2026-07-31",
      "type": "LoTW"
    },
    {
      "qso_id": 48190,
      "callsign": "K1ABC",
      "qso_date": "2026-07-25 10:12:00",
      "mode": "CW",
      "band": "2m",
      "gridsquare": "EM12",
      "sat_name": "AO-7",
      "sat_mode": "B",
      "confirmation_date": "2026-07-30",
      "type": "LoTW"
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 100,
    "count": 4,
    "total": 4,
    "total_pages": 1,
    "has_more": false,
    "filters": {
      "type": ["lotw", "eqsl"],
      "since": "2026-07-01",
      "qso_since": null,
      "qso_until": null,
      "band": null,
      "mode": null,
      "callsign": null
    }
  }
}
```

## Notes

- Rows are returned newest confirmation first (`confirmation_date`, then `qso_id`).
- The core fields (`qso_id`, `callsign`, `qso_date`, `mode`, `band`, `confirmation_date`, `type`)
  are always present. The optional fields — `submode`, `gridsquare`, `vucc_grids`, `sat_name`,
  `sat_mode` — are **omitted entirely** when empty, so the payload only carries what the QSO
  actually has. In the example above, the `OE3XYZ` rows have no optional fields, while the
  satellite contact carries `sat_name` / `sat_mode` / `gridsquare`.
- The `meta.filters` block echoes back the resolved filters, so a poller can interpret the rows
  without tracking the query it sent. `station_id` is resolved internally and is not echoed.
- Results are ordered newest-first; use [pagination](#filters) (`page` / `per_page`) and the
  `has_more` / `total_pages` meta to page through large result sets.

!!! note "Clubstation tokens are scoped to the member"
    Like the [QSO](qso.md) resource, a clubstation member below officer level only ever sees the
    confirmations of their **own operator's** QSOs. Officers see the whole club's confirmations.
