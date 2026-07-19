# QSL Queue

The QSL Queue is the hub for everything paper. QSOs that need a card end up here, and from here you print them as [labels](labels.md) or as [postcards](qsl-postcard-designer.md), export them, or mark them as sent.

Open it via `User menu -> QSL Queue`.

## How QSOs get into the queue

The queue holds every QSO whose **QSL sent** status is either `R` (Requested) or `Q` (Queued). There are several ways to get one there:

* Set **QSL sent** to *Requested* in the QSO form when adding or editing a QSO.
* Use the QSL buttons on a QSO row in the logbook.
* Mass-edit a selection in [Logbook Advanced](../logbook/advanced-logbook.md).
* Let [OQRS](oqrs.md) do it — a matched OQRS request is added to the queue automatically.

See the [status field reference](index.md#qsl-status-fields) for what the other values mean.

## The queue view

Filter by **station location** (or *All*) and choose whether the list shows the band, the frequency or both.

The **Previous QSL** column tells you how many QSLs have already been sent to the same station on the same band and mode — useful for deciding whether another card is really needed.

Select rows with the checkboxes, then use one of the four action menus.

### Mark

* **Mark selected QSOs as sent**
* **Mark ALL requested QSLs as sent** — applies to the whole current selection of stations, so use it deliberately

Marking as sent sets **QSL sent** to `Y` and stamps the QSL sent date. The QSO then leaves the queue.

### Export

* **Export selected QSOs to ADIF-file**
* **Export requested QSLs to ADIF-file**
* **Export requested QSLs to CSV-file**

The CSV export is meant for external label or card printing services. It contains the station callsign, the worked callsign, QSL via, date and time, mode, band, RST, the PSE/TNX QSL flag, comment, routing, ADIF DXCC entity and the gridsquares.

### Print

* **Print Selected QSO Postcards** / **Print Postcards for all QSOs** — opens the template picker of the [QSL Postcard Designer](qsl-postcard-designer.md)
* **Print Selected QSL Labels** / **Print Labels for all QSOs** — opens the print dialog of the [label](labels.md) engine

!!! note
    Printing does not mark anything as sent. Check the print result first, then use the **Mark** menu.

### Remove

**Remove selected QSOs from the queue** takes the QSOs out again without marking them as sent.

## QSO list and OQRS details

Each row offers a **QSO List** action showing all QSOs with that station that are pending — handy when you want to put several contacts on one card. If the entry came from OQRS, the original request including the requester's message is shown as well.
