# HRDLog.net

Wavelog pushes your QSOs to HRDLog.net. This is a **one-way** integration — HRDLog.net does not return confirmations.

The page lives at `User menu -> Third-Party Services -> HRDLog Logbook`.

## Setup

HRDLog.net is configured **per station location** (`Station Setup -> edit location`):

| Field | Description |
|---|---|
| **HRDLog.net Username** | The username you are registered with at HRDLog.net, usually your callsign |
| **HRDLog.net API Key** | The code from your HRDLog.net user profile |
| **HRDLog.net Logbook Realtime Upload** | *Yes* (push each QSO right after logging), *No* (batch only) or *Disabled* |

Create the API key on your <a href="https://www.hrdlog.net/EditUser.aspx" target="_blank" rel="noopener noreferrer">HRDLog.net user profile page</a>.

## Batch uploading

The batch upload catches QSOs that were edited after logging, or that failed while HRDLog.net was unreachable. Enable the `hrdlog_upload` job in the Cronmanager (`Admin -> Cron Jobs`); it defaults to every 6 hours.

You can also trigger an upload manually per station location on the HRDLog page.

## Marking QSOs as already uploaded

If you uploaded to HRDLog.net by other means before, use **Mark QSOs** on the HRDLog page to flag a date range as uploaded without sending it again. The same can be done during [ADIF import/export](../logbook/adif-import-export.md).
