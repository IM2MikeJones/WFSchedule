# WFSchedule

A single-page scheduling app with a Gantt-style timeline. You define tasks in a table (the source of truth) and view or edit them on an interactive canvas chart. No backend—runs entirely in the browser. Schedule state is stored in the URL hash so you can bookmark or share a link to restore it.

## Run

Open `index.html` in a modern browser (local file or any static server).

```bash
# Optional: serve locally
npx serve .
# or
python3 -m http.server 8080
```

Then open the URL your server shows (e.g. `http://localhost:8080`). Or open `index.html` directly via a file URL.

---

## App structure

Two main areas:

1. **Schedule (first card)** — Project name (optional), a collapse/expand toggle to the right of the name, add-row / clear-all, and a table of task rows. The table is the single source of truth; the chart reads from it. The card can be collapsed to show only the header row.
2. **Gantt chart (second card)** — Timeline view with day/week/month modes, weekend shading, optional dependency arrows, zoom, “Go to today,” settings panel, and a collapse/expand toggle to the right of the settings button. The card can be collapsed to show only the header row.

All editing is done in the table or by dragging bars on the chart (no separate form card).

---

## Schedule table (first card)

- **Project name** — Optional; no default. When set, shown in the chart header.
- **Add row** — Appends a new task row with defaults (see below). On first use the table is empty; the new row is the first. Subsequent rows default to linking to the previous row (From = Previous) with dates derived from the parent.
- **Clear all** — Confirmation dialog; clears all rows and reloads. Dialog offers “Copy URL to Save” before clearing.
- **Per row:**
  - **#** — Row number (read-only, display order).
  - **Properties** — T (task) / S (section), A, D, C (critical), M. Section rows hide most columns.
  - **ID** — Unique 3-character hex (e.g. `001`), stored in `data-row-id`; used for linking.
  - **Name** — Task or section name.
  - **Start date / End date** — Flatpickr date pickers. Changing start keeps duration and recalculates end; changing end recalculates duration; changing duration recalculates end. Same logic is used when dragging or resizing bars on the chart (one function: `applyStartEndToRow`).
  - **Progress** — 0–100; drives progress fill on the bar.
  - **Duration / Units** — Duration amount and unit (days/weeks/months). Changing units recalculates duration from current start/end.
  - **Delay** — Numeric delay (default 1). Can be negative (e.g. -999 to 999).
  - **Link** — Relationship type (Start to End, Start to Start, End to Start, End to End).
  - **From** — Target task for the link (—, Previous, Next, or row number). New rows default to Previous; first row has no default parent.
- **Row actions** — Delete row; drag handle to reorder. Reorder is reflected in the chart.
- **Persistence** — Changes are serialized into the URL hash (base64). Chart redraws immediately; URL update is debounced.

All row defaults are in JavaScript only (`ROW_DEFAULTS` and `applyRowDefaults`). The table starts empty; every added row gets these defaults. Row template for “Add row” comes from the first row once it exists, or from the built-in `#rowTemplate` when the table is empty.

---

## Linking (parent/child, Link / From / Delay)

Links are stored by **row ID**, not row number. The **From** dropdown is row-number-based (—, Previous, Next, 1, 2, 3, …); when the user changes From, the program resolves the selected row number to an ID using the current mapping and stores that ID. Row reorder only changes displayed row numbers and From options; it does not change which IDs are linked.

**Link type** (child→parent): Start to End, Start to Start, End to Start, End to End. If no parent (From = —), Link and Delay have no effect.

**Delay** is in the row’s **Units**. Delay 0 = child’s linked date = parent’s linked date (same day). Non-zero adds/subtracts that many units. When Units change, delay is recalculated in the new units (same actual time offset). Dates are always calendar days.

**Circular linking is not allowed.** Before applying a From/Link change, the program checks for a cycle; if so, the change is ignored and a warning is shown.

**Edit rules (current row = row being edited):**  
A1 Change Start → recalc delay, keep duration, recalc end, move progeny. A2 Change End → recalc duration, move progeny. A3 Change Duration → recalc end, move progeny. A4 Change Units → recalc duration and delay on current row only; no date changes. A5 Change Delay → recalc start, keep duration, recalc end, move progeny. A6 Change Link type → recalc start/end, move progeny. A7.1 From→other row → recalc start/end, move progeny. A7.2 From→— → break link only. A7.3 From — to parent → set start/end from parent+link+delay, move progeny. A8 Delete row → break link to parent and to immediate children only. B7 Task→Section (had parent and children) → break links; former parent adopts children; add this row’s delay to each child’s delay so no dates change. C1 Reorder → links stay by ID; only display recalc. C2 Insert / C3 Clear all / C4 No undo. D1 Drag bar = A1. D2 Resize bar = A2/A3.

**Future extensions:** Workdays vs calendar per row; “finish before” / drag front of bar to change start. See comments in code.

---

## Gantt chart (second card)

- **Header** — Project name from the first card (or empty). View mode (day/week/month), “Show weekends,” “Show dependencies,” zoom in/out, “Go to today,” settings toggle, and collapse/expand toggle (right of settings).
- **Settings panel** — Row height, bar height, bar corner radius, outline width, header height, name column width.
- **Canvas** — Bars show task span and progress. Click a bar to select; drag to move (updates table via `updateTableRowFromChart` → `applyStartEndToRow`); drag right-edge handle to resize. Dependency arrows when “Show dependencies” is on.
- **Weekend shading** — When “Show weekends” is on and view mode is day, Saturday/Sunday columns are shaded. Alpha is theme-dependent: light mode `0.4`, dark mode `0.05` (same RGB in both; constants `WEEKEND_RGB`, `WEEKEND_ALPHA` in code).
- **Bar colors** — Driven by row flags: **Active** → green, **Done** → gray, **Milestone** → blue; otherwise orange. **Critical** only changes the bar outline (red). The Color column is hidden in the table but still stored in data for future use.
- **Milestones** — Rows with `M` enabled are treated as milestones: they have **duration 0**, **end = start**, **delay -1**, and render as fixed-size diamond markers centered on their date boundary. They can be moved on the chart (drag) but not resized.
- **Data flow** — On each redraw the chart calls `getScheduleDataFromTable()` to build the task list. Rows without valid start/end dates are skipped. Table is the only source of truth.

---

## Tech

- **Single file** — `index.html` with inline CSS and two script blocks (theme + schedule table; Gantt chart).
- **Canvas** — Gantt drawing (no chart library). Chart theme (colors, weekend alpha) is built per draw from `data-theme` and constants.
- **Flatpickr** — Date inputs in the table.
- **Material Design 3** — Theming (tokens, outlined fields, sliders, dialog), light/dark mode (stored in `localStorage`).
- **No build step, no backend.**

---

## Version

Current revision is the main `index.html`.

- **Release snapshots** — Saved in folders like `Rev 1.0/` (and `Rev 1.1/` when you snapshot this version).
- **Release notes** — See `RELEASE_NOTES.md` (`v1.1` is the current release; `v1.2` is the next planned).

Older revisions are in `old revision 0.2`, `old revision 0.3 (problems with linking)`, etc., for reference.

---

## License

Use as you like. No warranty.
