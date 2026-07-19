# QO-100 Dx Club

Wavelog pushes your QSOs to the QO-100 Dx Club. This is a **one-way** integration — no confirmations are fetched back.

The page lives at `User menu -> Third-Party Services -> QO-100 Dx Club Upload`.

## Setup

The integration is configured **per station location** (`Station Setup -> edit location`):

| Field | Description |
|---|---|
| **QO-100 Dx Club API Key** | The key from your QO-100 Dx Club profile |
| **QO-100 Dx Club Realtime Upload** | *Yes* pushes each QSO right after logging, *No* leaves it to the manual/batch upload |

Create the API key on your <a href="https://qo100dx.club" target="_blank" rel="noopener noreferrer">QO-100 Dx Club profile page</a>.

!!! tip
    Only enable this on the station locations you actually use for QO-100. Locations without an API key are skipped.

## Uploading

Open the export page and pick a station location to upload everything that has not been sent yet. Successfully uploaded QSOs are marked, so a second run does not send them again.

## Scheduling

There is currently **no Cronmanager job** for the QO-100 Dx Club upload. Either use *Realtime* upload, run the manual upload when needed, or call the endpoint from your system crontab:

```bash
# Upload QSOs to the QO-100 Dx Club
21 */6 * * * curl --silent https://<url-to-wavelog>/index.php/webadif/upload &>/dev/null
```
