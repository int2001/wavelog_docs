# ADIF Import / Export

ADIF is the exchange format for amateur radio logs. Wavelog uses it to import logs from other programs — WSJT-X, other digimode software, another logger — and to export your log for third-party applications, awards or backups.

The page is reached via `User menu -> ADIF Import / Export` and is split into tabs: **ADIF Import**, **ADIF Export**, **DARC DCL**, **POTA** and **CBR Import**.

!!! note
    Large ADIF files take a while to process. Importing your whole ham career once is fine; doing it every day is not. For continuous WSJT-X logging there are better options — see <a href="https://github.com/wavelog/WaveLogGate" target="_blank" rel="noopener noreferrer">WaveLogGate</a> and the [WSJT-X integration](../integrations/wsjt-x.md).

## Importing

Pick the **station location** the QSOs belong to and choose your `.adi` file. Optionally assign the import to a contest.

!!! warning
    Files must have the type `*.adi`. The maximum upload size is shown on the page itself and is determined by your PHP configuration.

### Basic settings

| Option | Effect |
|---|---|
| **Import duplicate QSOs** | Import QSOs even if they already exist in the log |
| **Use DXCC information from ADIF** | Trust the DXCC data in the file. If unchecked, Wavelog determines the DXCC itself |
| **Always use the logged-in account callsign as the operator call** | Overrides the operator callsign from the file |

Two further options are marked **DANGER** because they switch off the safety checks that keep QSOs attached to the right station location:

* **Ignore Station callsign on import** — imports *all* QSOs from the file, regardless of whether they match the selected station location.
* **Ignore grid check on import** — imports QSOs even if their `MY_GRIDSQUARE` does not match the locator of the selected station location.

Only use these when you know exactly why the check fails. Otherwise you end up with QSOs filed under the wrong location, which quietly breaks awards and QSL uploads.

### Marking QSOs as uploaded

The **Mark QSOs as uploaded** section flags imported QSOs as already sent to LoTW, eQSL, HRDLog.net, QRZ, Clublog or DCL.

!!! warning
    This does **not** upload anything. Only tick a service if you have genuinely uploaded those QSOs there before — otherwise Wavelog will never send them, and the confirmations will never arrive.

Use it when the ADIF file itself does not carry that information, typically when migrating from another logger.

## Exporting

Choose a station location (or *All*) and optionally a date range, then export.

| Option | Effect |
|---|---|
| **Mark exported QSOs as already uploaded to LoTW** | Sets the LoTW sent flag on everything in the export |
| **Export QSOs not uploaded to LoTW** | Limits the export to QSOs still pending for LoTW |

Two shortcuts for satellite operators are available below the form: export all satellite QSOs, or only those confirmed on LoTW.

## Troubleshooting

If an import does not work as expected, see [ADIF import problems](../../troubleshooting/adif-cant-import.md).
