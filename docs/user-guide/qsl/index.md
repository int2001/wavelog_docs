# QSL Management Overview

Wavelog handles both sides of QSLing:

* **Electronic confirmations** — synchronising your log with online services such as Logbook of the World, eQSL.cc, Club Log, QRZ.com, HRDLog.net and the QO-100 Dx Club.
* **Paper QSL** — collecting QSL requests, queueing them and printing labels or full postcards.

This page explains the concepts that apply to all of them. The individual services and print workflows have their own pages.

## The one rule for all online services

!!! warning "Wavelog does not download your log"
    Wavelog **never** downloads a full log from a third-party service. A QSO must already exist in your Wavelog logbook. The services are only used to exchange **confirmations** for QSOs you have logged yourself.

    Once a confirmation for a logged QSO has been fetched, a green down-pointing arrow appears next to the QSO in the logbook.

If you want to get QSOs *into* Wavelog, use [ADIF import](../logbook/adif-import-export.md) or one of the [integrations](../integrations/wsjt-x.md) instead.

## Where to find things in the menu

| Menu | Entry | What it does |
|---|---|---|
| **Logbook** | View QSL Cards | Gallery of scanned paper cards you uploaded — see [Confirmations & Cards](confirmations.md) |
| **Logbook** | View eQSL Cards | Gallery of eQSL card images fetched from eQSL.cc |
| **Logbook** | View last confirmations | Filterable list of incoming confirmations across all services |
| **User menu** | OQRS Requests | Incoming [OQRS](oqrs.md) requests, with a badge showing open ones |
| **User menu** | QSL Queue | The [print queue](qsl-queue.md) for paper QSLs |
| **User menu** | Labels | [Label](labels.md) and paper type setup |
| **User menu** | QSL Postcard Designer | The [postcard designer](qsl-postcard-designer.md) |
| **User menu** | Third-Party Services | Submenu with LoTW, eQSL, HRDLog, QRZ, QO-100 Dx Club, Club Log and (if enabled) DCL |

## Service overview

| Service | Upload QSOs | Download confirmations | Card images | Configured in |
|---|---|---|---|---|
| [Logbook of the World](lotw.md) | yes (certificate) | yes (username/password) | no | User profile + certificate upload |
| [eQSL.cc](eqsl.md) | yes | yes | yes | User profile + station location nickname |
| [Club Log](clublog.md) | yes | yes | no | User profile |
| [QRZ.com Logbook](qrz-logbook.md) | yes | yes | no | Station location (API key) |
| [HRDLog.net](hrdlog.md) | yes | no | no | Station location (username + API key) |
| [QO-100 Dx Club](webadif.md) | yes | no | no | Station location (API key) |
| [DCL](../integrations/darc-dcl.md) | yes | not yet | no | Per-user DCL key, disabled by default |

Credentials live in **two different places** depending on the service:

* **User profile** (`User menu -> Edit profile -> Third Party Services`): LoTW, eQSL.cc and Club Log — one set of credentials for the whole account.
* **Station location** (`Station Setup -> edit location`): QRZ.com, HRDLog.net, QO-100 Dx Club and the eQSL QTH nickname — configured per location, because these services distinguish between your operating locations.

## QSL status fields

Every QSO carries a *QSL sent* and a *QSL received* status. These refer to the **paper** QSL; each online service tracks its own status separately.

### QSL sent

| Value | Meaning |
|---|---|
| `N` | No — nothing sent |
| `Y` | Yes — card sent |
| `R` | Requested — someone asked for a card; the QSO appears in the [QSL Queue](qsl-queue.md) |
| `Q` | Queued — the QSO was put into the queue for printing |
| `I` | Ignore — the QSO is excluded from QSLing |

### QSL received

| Value | Meaning |
|---|---|
| `N` | No |
| `Y` | Yes — card received |
| `R` | Requested |
| `I` | Invalid (ignore) |
| `V` | Verified (match) |

!!! note
    Only `R` and `Q` put a QSO into the QSL Queue. Everything that gets printed from the queue is set to `Y` afterwards.

### Where you set them

* **QSO form** — the *QSL* section when adding or editing a QSO, including the *QSL via* and *QSL message* fields.
* **Logbook grid** — the small QSL buttons on a QSO row set *received*, *sent*, *requested* or *ignore* directly, including the method used (bureau, direct, ...).
* **[Logbook Advanced](../logbook/advanced-logbook.md)** — mass edit for changing many QSOs at once.
* **[ADIF import/export](../logbook/adif-import-export.md)** — the *mark as uploaded to ...* checkboxes.
* **The sync jobs themselves** — each service marks its own QSOs as sent/confirmed on success.

## Scheduling the sync jobs

All up- and downloads run as background jobs. Manage them in **Admin -> Cron Jobs** (the Cronmanager), where each job can be enabled, rescheduled and run manually:

| Job | Purpose | Default schedule |
|---|---|---|
| `lotw_lotw_upload` | Upload to and download from LoTW | hourly |
| `eqsl_sync` | Upload to and download from eQSL.cc | every 6 h |
| `clublog_upload` | Upload to Club Log | every 6 h |
| `qrz_upload` / `qrz_download` | QRZ.com Logbook | every 6 h |
| `hrdlog_upload` | Upload to HRDLog.net | every 6 h |
| `sync_dcl` | Upload to DCL | daily, disabled |

All of these are **disabled by default** — enable the ones you need.

!!! note
    The Cronmanager itself needs to be triggered by the system cron of your server. See [Cron Jobs](../../admin-guide/administration/cron-jobs.md) for the setup.
