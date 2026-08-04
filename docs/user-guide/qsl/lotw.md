# Logbook of the World

Wavelog uploads your QSOs to the ARRL **Logbook of the World** (LoTW) and downloads the resulting confirmations. Both directions are handled by the same background job.

The page lives at `User menu -> Third-Party Services -> Logbook of the World`.

!!! warning "Wavelog does not download your log"
    Only **confirmations** are fetched from LoTW. The QSO itself must already be in your Wavelog logbook — see the [overview](index.md).

Uploading and downloading use **different credentials**:

* **Upload** is signed with your LoTW **certificate** (`.p12`). No username or password involved.
* **Download** of confirmations uses your LoTW **username and password**.

You need both configured for a complete sync.

## Setting up credentials

Go to `User menu -> Edit profile -> Third Party Services` and fill in your LoTW username and password. Use the **Test Login** button next to the fields to verify them right away.

!!! note
    The credentials are stored **per user account**, not per station location. The same login is used for every callsign in that account. If you hold several LoTW accounts for different callsigns, ask the [LoTW Helpdesk](https://www.arrl.org/lotw-help-ticket) to merge them — a single LoTW account can handle multiple callsigns.

    For US 1x1 callsigns, use the login of the primary account that requested the certificate for the 1x1; there is no separate LoTW login for the 1x1 itself.

## Uploading your certificate

Export the certificate from TQSL (version 2.7 or later):

1. Open TQSL and go to the **Callsign Certificates** tab.
2. Right-click the callsign you want to export.
3. Choose **Save Callsign Certificate File** and **do not set a password**.

Then upload the resulting `.p12` file in Wavelog under `Logbook of the World -> Upload certificate`.

!!! warning
    A password-protected `.p12` cannot be imported. Wavelog needs to read the private key to sign uploads unattended.

Once uploaded, the certificate list shows:

| Column | Meaning |
|---|---|
| Callsign | The callsign the certificate was issued for |
| DXCC | The DXCC entity of the certificate |
| Date Created | When the certificate was issued |
| Date Expires | When the certificate itself expires |
| Status | Badges for validity, revocation state and the result of the last upload |

Wavelog checks the ARRL revocation list every time this page is opened, so a certificate that was superseded at LoTW is flagged here.

!!! tip
    If the upload of the `.p12` fails with an OpenSSL-related error, the PHP `openssl` extension is probably not enabled. See [LoTW P12 File – Not Possible to Upload](../../troubleshooting/lotw-p12-upload.md).

### Certificate validity vs. QSO date range

A LoTW certificate carries **two independent date ranges**:

* The **certificate validity** — the period during which the certificate can be used to sign logs at all.
* The **QSO start and end date** — the period the QSOs themselves must fall into.

Example: a certificate valid from `2025-01-01` to `2030-12-31` with a QSO range of `2025-02-23` to `2026-08-28` cannot sign a QSO made in January 2025 or in September 2026, even though the certificate is valid until 2030.

Unless you are documenting a limited operation such as a DXpedition or a deleted DXCC entity, leave the **QSO end date empty** when requesting the certificate:

<img width="698" height="514" alt="QSO end time field in TQSL" src="https://github.com/user-attachments/assets/797399ea-c96d-4553-8439-3b1407cb5234" />

### Server-side storage of the keys

The private keys are not kept in the web root. The directory Wavelog uses can be configured in `application/config/lotw.php`. Choose a location outside the document root and protect it on the file system, as ARRL's guidance requires.

## Uploading QSOs

Wavelog builds the signed TQ8 file **per station location**, using the data from the location itself. If you operate from a new place, just create the station location — the upload job picks it up automatically and checks whether a matching certificate exists.

A QSO is skipped when the station location has no certificate, the DXCC does not match, or the certificate is expired, superseded or outside its QSO date range.

The upload runs as the `lotw_lotw_upload` job in the Cronmanager (`Admin -> Cron Jobs`), by default hourly. Do not schedule it more often than every hour or two.

### Regional notes

**United States** — the county in the station location must be spelled exactly as LoTW expects, for example state `OR` and county `Deschutes`.

**Canada** — select your province via the state drop-down in the station location.

## Downloading confirmations

There are two ways to get matches from LoTW:

* **Direct download** — Wavelog fetches the matches itself using your LoTW username and password. This happens as part of the same `lotw_lotw_upload` job.
* **ADIF file** — export a report from the LoTW website and upload it on the LoTW import page. Follow the instructions on that page to generate the correct file.

## FAQ

### Why does the LoTW confirmation badge in the logbook show outdated data?

The "last upload to LoTW" information for a DX station comes from the `lotw-user-activity.csv` file published by LoTW. ARRL updates it roughly once a week, around 1000z on Sundays, and Wavelog's `update_lotw_users` job imports it weekly — so the data can be up to a week old.

For the current state of a single callsign, use [Logbook Find Call](https://lotw.arrl.org/lotwuser/act?awg_id=&ac_acct=) or the **Last Upload** link in the QSO details. Both require a logged-in LoTW account.

### The upload succeeded but LoTW says the import failed — why are the QSOs marked as sent?

Upload and processing at LoTW happen asynchronously. Wavelog only sees whether the *file transfer* succeeded and marks the QSOs as sent at that point. See [Known issues](#known-issues) below.

### Uploads work but downloads do not

Check your LoTW username and password. Uploads only need the certificate, downloads need the login. If you changed your password at LoTW but not in Wavelog, uploads keep working while downloads fail.

The typical symptom during a manual sync is: `Downloaded LoTW report contains no matches`.

### Some QSOs still refuse to upload

Check that the QSO date falls within the QSO start and end date of the certificate — see [above](#certificate-validity-vs-qso-date-range). QSOs outside that window cannot be signed and need a new certificate with a suitable range.

Another common cause is a **superseded certificate**: if you renewed before expiry, the old certificate is still valid in Wavelog but rejected by LoTW. Replace it with the new one.

### Why doesn't Wavelog fetch older confirmations?

Wavelog only asks LoTW for confirmations *newer* than the last one it received. The reasoning and the workaround are the same as for QRZ.com — see [Sync logic](qrz-logbook.md#how-the-sync-logic-works).

## Known issues

### No feedback on rejected QSOs

If ARRL does not accept part of the uploaded file, the QSOs are still marked as *LoTW sent* in Wavelog, because the automatic upload provides no per-QSO feedback. This cannot be fixed from Wavelog's side.

To check, log in at the LoTW website and go to **Your Account -> Your Activity**. To retry a corrected QSO, clear its *LoTW sent* field manually.
