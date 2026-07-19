# eQSL.cc

Wavelog synchronises your log with eQSL.cc in both directions and caches the received card images locally, so you can handle your eQSL business without logging into eQSL.cc.

The page lives at `User menu -> Third-Party Services -> eQSL Import / Export`.

!!! warning "Wavelog does not download your log"
    Only **confirmations** are fetched from eQSL.cc. The QSO itself must already be in your Wavelog logbook — see the [overview](index.md).

## Common pitfalls

eQSL.cc has a number of quirks that cause most of the support questions. Read these before setting up:

!!! warning "Read this first"
    - **Every location is its own account.** eQSL treats each _Nickname_ (QTH) as a separate account with its own credentials. If you change the password of your default QTH, the other nicknames keep the old one — change **every** password.
    - **Wavelog deletes your credentials on authentication errors.** If eQSL answers _wrong password / user not found_, Wavelog removes the stored credentials to prevent eQSL from blocking your account or the whole instance. Details end up in the application log (`application/logs`, log level in `application/config/config.php`).
    - **No overlapping QTHs.** Each nickname needs its own validity period. Uploading a QSO outside the date range of a nickname produces an error — which can in turn trigger the credential removal above.
    - **No special characters in the password.** The eQSL front end accepts them, the API does not.
    - **Keep the password short.** Long passwords (25 characters and more) are accepted by the eQSL website but rejected by their API. Around 8 plain characters works reliably.

## Setup

1. Enter your eQSL.cc username and password under `User menu -> Edit profile -> Third Party Services`. Your eQSL username is the callsign of that account.
2. For **each station location** you use, set the matching **eQSL QTH Nickname** (`Station Setup -> edit location`). Using the same station locator on both sides is a good way to keep them apart.
3. Make sure the QSOs you want to upload fall within the start/end date range and use the callsign configured for that nickname at eQSL.

Optionally, set an **eQSL default QSL message** per station location. It pre-fills the eQSL message field when you add a QSO from that location.

## Uploading QSOs

**Upload QSOs** shows the list of QSOs that have not been sent to eQSL yet. Check the list and confirm — successfully uploaded QSOs are marked as sent in the database.

## Importing confirmations

On the eQSL page you have two options:

* **Pull eQSL for me** — Wavelog downloads all your latest matches from eQSL and marks them as received.
* **Upload an ADIF file** — import an inbox ADIF exported from eQSL.cc.

## Viewing the cards

Clicking the received arrow of a confirmed QSO in the logbook downloads the eQSL card image and caches it on your server. All cached cards are collected under `Logbook -> View eQSL Cards` — see [Confirmations & Cards](confirmations.md#view-eqsl-cards).

## Tools

**Mark all QSOs as sent to eQSL** marks every pending QSO as uploaded without actually sending it. This is what you want on a fresh install where you already uploaded thousands of QSOs manually.

## Scheduling

Enable the `eqsl_sync` job in the Cronmanager (`Admin -> Cron Jobs`). It handles upload and download in one run and defaults to every 6 hours.

## Debugging

If uploads fail, raise the log level in `application/config/config.php` (`log_threshold`) and check the files in `application/logs`.
