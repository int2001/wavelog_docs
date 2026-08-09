# Activation Planner

The **Activation Planner** is a full-page planning map for portable, SOTA, WWFF and POTA operators. Type a Maidenhead gridsquare and the map zooms straight to it, draws a box around the exact grid cell, and shows you the surrounding squares, reference directories, zones and administrative regions — everything you need to pick a spot before you head out.

---

## Opening the page

From the Wavelog menu, choose **Activation Planner** (the map-marked icon), found in the TOOLS menu group.

---

## The toolbar

The toolbar at the top holds every control. On a phone the secondary controls are tucked behind a **More options** button; tap it to reveal the second gridsquare box and the overlay dropdowns.

| Control | What it does |
| --- | --- |
| **Gridsquare** | Type a Maidenhead locator here (2, 4, 6, 8 or 10 characters, e.g. `FN`, `FN31`, `FN31pr`). Press **Enter** or click **Go**. |
| **to** (second box) | Optional. Enter a second gridsquare to measure distance and bearing between the two (a QRB calculation). Leave empty for a single-grid lookup. |
| **Go** | Looks up the entered grid(s), zooms in, and fills the info bar. |
| **Clear** | Removes every marker, square and line, stops GPS tracking, and zooms back out to the world view. |
| **Locate me** | Asks your device for its GPS position, fills the gridsquare box, and starts live tracking. (See [Find my location](#find-my-location-gps).) |
| **Refs** dropdown | Toggles the gridsquare overlay and the **WWFF / POTA / SOTA** reference directories. Each item carries a coloured letter badge (**W** / **P** / **S**) matching its markers; the button turns yellow while any item is on. |
| **Overlays** dropdown | Toggles **CQ Zones**, **ITU Zones** and per-country **States / Provinces** boundaries. The button turns yellow while any overlay is on, and the map zooms to fit an overlay when you enable it. |

> **Tip:** the collapsible card header ("Plan your activity here") can be clicked to collapse the toolbar and give the map more room.

---

## Single gridsquare lookup

1. Type a locator in the **Gridsquare** box (e.g. `JN58qm`).
2. Click **Go**.

The map zooms to that grid and:

- Draws a **box** around the exact cell.
- Drops a **marker** at the cell centre.
- Fills the **info bar** with the locator, the cell size label (Field / Grid square / Subsquare …), its coordinates, the **CQ and ITU zones**, and the **state/province** it falls in.
- Opens the **surrounding-squares readout** (see below).

### Surrounding-squares readout

When a point is selected, the planner shows a 3×3 compass of the eight neighbouring grids (N, NE, E, SE, S, SW, W, NW) around the centre square. Each cell names the grid you'd step into and gives the distance and bearing to that edge of your square; the **nearest** border is highlighted. On the map itself, matching direction arrows point to each border with inline distance labels — so the information stays visible even on a phone, where the corner panel is hidden.

The red/green arrows and the square outline can be cleared independently: the panel's ✕ button hides the legend but keeps the on-map arrows until you press **Clear**.

---

## Two-gridsquare QRB (distance & bearing)

1. Type the **first** gridsquare.
2. Type a **second** gridsquare in the **to** box.
3. Click **Go**.

The planner draws both squares (blue and orange), plots a dashed **great-circle path** between them, and reports:

- The **distance** between the two cell centres.
- The **bearing** from grid 1 → grid 2 (in degrees and as a compass point, e.g. `074° (ENE)`).
- Each grid's CQ/ITU zones and state/province, appended as it resolves.

Distance follows your **measurement base** setting — kilometres (K), miles (M) or nautical miles (N).

---

## The gridsquare popup

Clicking the map or looking up a grid drops a marker whose popup is built around the locator:

- a red left stripe with a **Gridsquare** header pill;
- the **DXCC flag(s)** for the 4-character grid (resolved from the VUCC grids table), shown just before the locator — a gridsquare that spans two or more countries, shows all flags;
- the **coordinates**;
- the **Field / Square / Subsquare** hierarchy of the locator;
- the resolved **CQ Zone** and **ITU Zone** on one line, with the **state/province** on the next;
- a **Satellite passes** link that opens the pass-prediction page with this grid pre-filled;
- quick actions: share the spot on **X** or **bluesky**, or **create a station location** here (gridsquare, DXCC (only if gridsquare exists in one DXCC), CQ zone, ITU zone and any SOTA/POTA/WWFF refs pre-filled);
- any **WWFF / POTA / SOTA references** that fall inside the grid.

---

## Find my location (GPS)

Click **Locate me** to use your device's GPS:

- Your current position is converted to a gridsquare and dropped on the map exactly as if you'd typed it.
- The gridsquare is also filled into the **Gridsquare** box, so you can immediately type a second grid and hit **Go** for a QRB *from your current location*.
- The button switches to a green **Tracking** state and polls your position roughly every 60 seconds, so the marker follows you while you move. Map clicks are ignored while tracking, so your position stays centred; press **Clear** to stop tracking and re-enable click-selection.

### Requirements & errors

- Geolocation requires a **secure context**: the page must be served over **HTTPS**, or from **`localhost`**. Over plain HTTP on a LAN IP the button is hidden automatically.
- The first time you use it, your browser will ask permission. If you deny it, a "Location access denied" toast is shown and tracking stops.
- Transient failures (position unavailable / timed out) show a warning toast and the poll continues.

---

## Reference directories (WWFF / POTA / SOTA)

Open the **Refs** dropdown to plot the full reference directories on the map. Each menu item carries a coloured letter badge — **W** (blue) for WWFF, **P** (green) for POTA, **S** (orange) for SOTA — the same badge drawn on its markers.

| Directory | Badge | Shows |
| --- | --- | --- |
| **WWFF** | W (blue) | Every World Wide Flora & Fauna reference with coordinates |
| **POTA** | P (green) | Every Parks on the Air reference |
| **SOTA** | S (orange) | Every Summits on the Air summit |

Each reference is drawn as a small **lettered circle** in its directory colour and clustered when dense. There are no hover tooltips — **click** a marker for a rich popup:

- a coloured left stripe and a header with a type pill (**WWFF** / **POTA** / **SOTA**) beside the **reference**, which links to that reference's info page ([gma.rocks](https://www.gma.rocks) for WWFF, [pota.app](https://pota.app) for POTA, [sotadata.org.uk](https://www.sotadata.org.uk) for SOTA) and opens in a new tab;
- the summit / area **name**;
- the summit **altitude** (SOTA only), shown in your measurement unit — metres for Kilometres users, feet otherwise (the database stores metres);
- the **gridsquare** and **coordinates**.

The directories are large, so they load **lazily on first toggle**: a spinner covers the map while the data downloads and the markers are placed, then the directory is cached for instant re-toggle.

The **Gridsquare** checkbox in the same menu toggles the Maidenhead grid overlay drawn across the map (its badge is red, matching the grid lines).

---

## Zone & region overlays (GeoJSON)

Open the **Overlays** dropdown to draw boundaries from GeoJSON files. Items are grouped:

- **Zones** — **CQ Zones** and **ITU Zones** (worldwide).
- **States / Provinces** — administrative subdivisions, one entry per supported country.

Each overlay is fetched once and cached. When you turn one on, the map **zooms to fit it**. Features carry **permanent labels** — the zone number for CQ/ITU zones, the state/province name for subdivisions — so you can read them at a glance. The overlays are **non-interactive**: they don't capture clicks or hovers, so you can still click straight through them to select a point on the map.

When you select a point or type a grid, the **CQ zone**, **ITU zone** and **state/province** it falls in are resolved and shown in the info bar — even if the corresponding overlay is not turned on.

### Supported States / Provinces

Subdivision overlays are available for (among others): Argentina, Australia, Azores, Belarus, Belgium, Brazil, Bulgaria, Canada, Canary Islands, Ceuta & Melilla, Chile, China, Corsica, France, Germany, Hawaii, Hungary, India, Ireland, Italy, Japan, Madeira Islands, Mexico, Netherlands, New Zealand, Norway, Papua New Guinea, Poland, Portugal, Republic of Korea, Romania, Sardinia, Spain, Sweden, Switzerland, USA, Uruguay, Venezuela, and Alaska. The list grows as more GeoJSON files are added.

---

## The coordinate bar

As you move the mouse over the map, a bar below the map updates live with:

- **Latitude** / **Longitude** (in DMS),
- the **Gridsquare** under the cursor,
- the **CQ Zone** and **ITU Zone** under the cursor.

These are read-only live readouts; clicking the map selects a point and fills the info bar as described above.

---

## Notes & tips

- **Locator precision:** the box is drawn for whatever precision you type — a 2-character *Field* (large), a 4-character *Grid square*, a 6-character *Subsquare*, right down to 8- and 10-character cells.
- **Click instead of type:** you can click anywhere on the map to drop a marker and get the same gridsquare, zone, region and surrounding-squares info.
- **Dark themes:** the selected square is drawn yellow on dark themes and red on light themes for best contrast.
- **Fullscreen:** use the fullscreen control on the map for an uncluttered planning view.
- **Performance:** zones and directories load on demand and are cached; a spinner covers the map while a reference directory downloads and its markers are placed, so the page stays light and responsive unless you actively turn overlays on.
