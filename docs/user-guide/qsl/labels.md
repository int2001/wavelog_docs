# Labels

Wavelog prints QSL labels as a ready-to-print PDF, for both label sheets and dedicated label printers. Everything is driven from `User menu -> Labels`.

Labels are printed from the [QSL Queue](qsl-queue.md), so make sure your QSOs are marked as *Requested* or *Queued* first.

## The two building blocks

Wavelog ships no presets — you define your own:

* A **paper type** describes the sheet or the endless roll: name, unit, width, height and orientation.
* A **label type** describes the individual labels on that paper: how many fit horizontally and vertically, their size, the margins, the spacing and the font.

Create the paper type first, then the label type that references it.

## Creating a paper type

`Labels -> Create Paper Type`. Give it a name, choose millimeters or inches, and enter the physical size and orientation of the sheet.

## Creating a label type

`Labels -> Create New Label Type`:

| Field | Meaning |
|---|---|
| Label Name | Free text — use something recognisable, e.g. the label article number |
| Paper Type | The paper type created above |
| Measurement used | Millimeters or inches |
| Margin Top / Margin Left | Distance from the paper edge to the first label |
| Labels horizontally / vertically | The grid of labels on the sheet |
| Horizontal / Vertical space | Gap between two labels |
| Width of label / Height of label | Size of a single label |
| Font Size | Point size used on the label — don't go too big |
| QSOs on label | How many QSOs are grouped onto one label |

Save it, then click **Use For Print** so Wavelog knows which label type to use.

## Printing

Go to the [QSL Queue](qsl-queue.md), select the QSOs and choose **Print Selected QSL Labels** (or *Print Labels for all QSOs*). The print dialog offers:

* **Start at label** — if you already used part of a sheet, pick the first free position so nothing is wasted.
* **Include gridsquare** — print your locator.
* **Include via** — print the QSL route (*via* callsign).
* **QSL message** — print the per-QSO QSL message.
* **PSE/TNX message** — print whether you are asking for a card or thanking for one.
* **References** — print SIG, SOTA, IOTA, POTA and WWFF references.
* **My callsign** / **Operator callsign** — print the station and/or operator callsign.

QSOs with the same callsign are grouped onto one label, up to the *QSOs on label* limit you configured. Satellite QSOs are grouped by satellite mode as well.

!!! tip
    Use your operating system's print dialog rather than the browser's. Browsers tend to ignore the landscape/portrait setting of the generated PDF.

## Example: Brother label printer with 38 × 90 mm labels

Printing on a dedicated label printer takes a bit of care.

**1. Create the paper type.** `Labels -> Create Paper Type`, filled in like this:

<img width="1308" alt="Paper type dialog for a Brother label printer" src="https://github.com/wavelog/wavelog/assets/1410708/a55d2b8d-f9c7-417a-9d4d-240f7b38a776">

**2. Create the label type** and select the paper type you just created.

Note that width and height look swapped — that is intentional. Brother printers sometimes insist on portrait mode; otherwise they refuse to print.

<img width="1312" alt="Label type dialog for a Brother label printer" src="https://github.com/wavelog/wavelog/assets/1410708/efb92f16-cb9c-4d8e-8d8e-d507b9906e83">

**3. Set it as the label to use for print**, then print from the QSL Queue as described above.

## Labels or postcards?

Labels are the classic route: you print an address-and-QSO label and stick it onto a pre-printed card. If you would rather print the whole card, including a background image, use the [QSL Postcard Designer](qsl-postcard-designer.md) instead. Both are fed from the same queue.
