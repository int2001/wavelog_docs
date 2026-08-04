# QRZ.com Logbook

Wavelog pushes your QSOs to the QRZ.com Logbook and downloads the confirmations QRZ holds for you.

The page lives at `User menu -> Third-Party Services -> QRZ Logbook`.

!!! note "Requirements"
    Using the QRZ Logbook API requires a **paid QRZ subscription**.

!!! warning "Wavelog does not download your log"
    Only **confirmations** are fetched from QRZ.com. The QSO itself must already be in your Wavelog logbook — see the [overview](index.md).

## Setup

QRZ is configured **per station location** (`Station Setup -> edit location`):

| Field | Description |
|---|---|
| **QRZ.com Logbook API Key** | The key for the logbook this location should upload to. Format: `XXXX-XXXX-XXXX-XXXX` |
| **QRZ.com Logbook Upload** | *Disabled*, *Enabled* (batch upload only) or *Realtime* (push each QSO right after logging) |

Find your API key at [logbook.qrz.com/logbook](https://logbook.qrz.com/logbook). Each QRZ logbook has its own key, so a location that logs under a different callsign needs the key of the matching logbook.

!!! warning
    If QRZ rejects the key repeatedly (`STATUS=AUTH`), Wavelog sets that location's upload to *Disabled* to avoid hammering the API. Fix the key and re-enable it.

## Batch up- and download

Even with *Realtime* enabled, the batch jobs are useful: they catch up QSOs that were edited afterwards or that failed while QRZ was unreachable.

Enable the jobs in the Cronmanager (`Admin -> Cron Jobs`), both default to every 6 hours:

* `qrz_upload` — send new and changed QSOs to QRZ
* `qrz_download` — fetch confirmations

## Marking QSOs as already uploaded

If you uploaded to QRZ by other means before, you can mark a date range as uploaded without sending it again — either on the QRZ page or via the *mark as uploaded* checkboxes during [ADIF import/export](../logbook/adif-import-export.md).

## How the sync logic works

Wavelog asks QRZ for confirmations **newer than the last one it already received**. An example:

* A QSO is confirmed on 2023-01-01 — either really confirmed, or marked as confirmed by you via the UI or an ADIF import.
* From then on, Wavelog only asks QRZ for confirmations dated **after** 2023-01-01.

A confirmation dated *before* that will therefore never arrive.

Why it works this way:

* It saves bandwidth and computing power on both sides.
* An already confirmed QSO does not need to be confirmed again.
* QRZ processes confirmations chronologically. You either get a confirmation *now* for a past QSO, or you get it in the future — you never receive one whose date lies in the past.

The same logic applies to [LoTW](lotw.md).

### Workaround

If you are convinced confirmations are missing, mark **all** your QSOs as *not confirmed via QRZ*. The next sync then starts from scratch. You can do this by:

* using the mass edit in [Logbook Advanced](../logbook/advanced-logbook.md) — the recommended way,
* editing the QSOs one by one, or
* exporting the log, clearing it, editing the ADIF in an editor and importing it again.
