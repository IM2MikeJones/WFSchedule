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

Four cards, top to bottom:

1. **Top card** — Always visible (no collapse control). Displays the project name as a read-only title, synced from the schedule card's Project Name input.
2. **Schedule card** — Collapse/expand arrow at top-right. When expanded: project name input, add-row / clear-all, and a table of task rows. The table is the single source of truth; the chart reads from it. When collapsed, only the toggle is visible. Hidden by default (requires admin char `e`/`E` in URL — see [Admin char](#admin-char)).
3. **Gantt chart card** — Gear icon (settings) and collapse/expand arrow in the top bar. When expanded: settings panel (toggled by gear icon), and the timeline canvas. When collapsed, only the top-bar controls are visible.
4. **Text schedule card** — Copy the full schedule as a monospace-aligned text block (fields separated by `^`), or paste a block to replace the entire table. Only paste is accepted in the textarea; typing is disabled. The `^` character is reserved and will be stripped / warned about if used in project, task, or table fields. Legacy tab-separated blocks still import.

All editing is done in the table or by dragging bars on the chart (no separate form card).

---

## Admin char

A single character at the start of the URL hash controls which cards are visible:

- **No admin char** (e.g. `yourpage.html#` or `yourpage.html#<payload>`): Only the chart and top cards are visible. The schedule (table) card is hidden. Existing bookmarks without a prefix continue to work.
- **Admin char `e` or `E`** (e.g. `yourpage.html#e` or `yourpage.html#e<payload>`): Both the schedule and chart cards are visible.

The admin char is preserved automatically when the URL is updated. To change visibility, edit the character at the start of the hash in the address bar and refresh.

---

## Schedule table

- **Project name** — Optional; no default. When set, shown in the top card title and in the browser tab title. Saved in the URL for bookmarks.
- **Add row** — Appends a new task row with defaults (see below). On first use the table is empty; the new row is the first. Subsequent rows default to linking to the previous row (From = Previous) with dates derived from the parent.
- **Clear all** — Confirmation dialog; clears all rows and reloads. Dialog offers "Copy URL to Save" before clearing.
- **Per row:**
  - **#** — Row number (read-only, display order).
  - **Properties** — T (task) / S (section), plus status buttons showing the first letter of each custom label (Active, Done, Critical, Milestone). Section rows hide most columns.
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

All row defaults are in JavaScript only (`ROW_DEFAULTS` and `applyRowDefaults`). The table starts empty; every added row gets these defaults. Row template for "Add row" comes from the first row once it exists, or from the built-in `#rowTemplate` when the table is empty.

---

## Linking (parent/child, Link / From / Delay)

Links are stored by **row ID**, not row number. The **From** dropdown is row-number-based (—, Previous, Next, 1, 2, 3, …); when the user changes From, the program resolves the selected row number to an ID using the current mapping and stores that ID. Row reorder only changes displayed row numbers and From options; it does not change which IDs are linked.

**Link type** (child→parent): Start to End, Start to Start, End to Start, End to End. If no parent (From = —), Link and Delay have no effect.

**Delay** is in the row's **Units**. Delay 0 = child's linked date = parent's linked date (same day). Non-zero adds/subtracts that many units. When Units change, delay is recalculated in the new units (same actual time offset). Dates are always calendar days.

**Circular linking is not allowed.** Before applying a From/Link change, the program checks for a cycle; if so, the change is ignored and a warning is shown.

**Edit rules (current row = row being edited):**  
A1 Change Start → recalc delay, keep duration, recalc end, move progeny. A2 Change End → recalc duration, move progeny. A3 Change Duration → recalc end, move progeny. A4 Change Units → recalc duration and delay on current row only; no date changes. A5 Change Delay → recalc start, keep duration, recalc end, move progeny. A6 Change Link type → recalc start/end, move progeny. A7.1 From→other row → recalc start/end, move progeny. A7.2 From→— → break link only. A7.3 From — to parent → set start/end from parent+link+delay, move progeny. A8 Delete row → break link to parent and to immediate children only. B7 Task→Section (had parent and children) → break links; former parent adopts children; add this row's delay to each child's delay so no dates change. C1 Reorder → links stay by ID; only display recalc. C2 Insert / C3 Clear all / C4 No undo. D1 Drag bar = A1. D2 Resize bar = A2/A3.

**Future extensions:** Workdays vs calendar per row; "finish before" / drag front of bar to change start. See comments in code.

---

## Gantt chart

- **Settings panel** (gear icon in top bar) — Two tiers:
  - **Settings**: Show weekends, Show dependencies, Day/Week/Month view mode, Zoom in/out, Go to today, and Advanced button (floated right).
  - **Advanced**: Accent theme (6 options), row height, bar height, bar corner radius, outline width, header height, and status color/label buttons (Default, Active, Done, Critical, Milestone) that open dialogs with color pickers and optional custom labels (14 colors each, saved in the URL).
- **Header** — Top-left corner labels: day="Month - Year"/"Date", week="Month - Year"/"Week Number", month="Year"/"Month". In week mode, the year is shown only when it changes (first month and each January).
- **Canvas** — Bars show task span and progress. Click a bar to select; drag to move (updates table via `updateTableRowFromChart` → `applyStartEndToRow`); drag right-edge handle to resize. Dependency arrows when "Show dependencies" is on.
- **Weekend shading** — When "Show weekends" is on and view mode is day, Saturday/Sunday columns are shaded. Uses `--wf-chart-weekend` from the chart theme (theme-dependent).
- **Bar colors** — Driven by row flags. Defaults: Default=gray, Active=green, Done=slate, Critical=red (outline only), Milestone=teal. Each status has 14 color options. The Color column is hidden in the table but still stored in data for future use.
- **Milestones** — Rows with `M` enabled are treated as milestones: they have **duration 0**, **end = start**, **delay -1**, and render as fixed-size diamond markers centered on their date boundary. They can be moved on the chart (drag) but not resized.
- **Data flow** — On each redraw the chart calls `getScheduleDataFromTable()` to build the task list. Rows without valid start/end dates are skipped. Table is the only source of truth.

---

## Tech

- **Single file** — `index.html` with inline CSS and two script blocks (theme + schedule table; Gantt chart).
- **Canvas** — Gantt drawing (no chart library). Chart theme (colors, weekend shading) is built per draw from CSS variables (`--wf-chart-*`) and `data-theme`.
- **Flatpickr** — Date inputs in the table.
- **Material Design 3** — Theming (tokens, outlined fields, sliders, dialog), light/dark mode (stored in `localStorage`), six accent themes (blue, green, purple, teal, orange, rose) in settings.
- **No build step, no backend.**

---

## Version

Current version: **v1.5**. Changes per release are documented in `RELEASE_NOTES.md`.

> **Compatibility note (v1.2)**
> URL encoding changed to a compact format. Links from v1.0/v1.1 will not load in v1.2+; v1.2+ URLs are not for older builds.

- **Release notes** — See `RELEASE_NOTES.md`.

Older revisions are in `old revision 0.2`, `old revision 0.3 (problems with linking)`, etc., for reference.

---

## License

Use as you like. No warranty.
