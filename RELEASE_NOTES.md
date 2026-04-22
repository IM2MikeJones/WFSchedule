# Release notes — WFSchedule

This file tracks changes **going forward**.

- **How you use it**: when you decide a release is “done”, you create a new `Rev X.Y/` folder, move `index.html` + docs (including this file) into it, then tell me to start a fresh **Unreleased** section for the next version.

---

## Unreleased

_(start the next session here)_

---

## v1.5 — Released

### Added

- **Admin char (URL visibility control)** — A single-character prefix in the URL hash controls which cards are visible:
  - **No admin char** (e.g. `#` or `#<base64-payload>`): Only the chart card is visible. The schedule (table) card is hidden. Existing bookmarks without a prefix continue to work.
  - **Admin char `e` or `E`** (e.g. `#e` or `#e<base64-payload>`): Both the schedule and chart cards are visible.

  **How to use:**
  1. To show the schedule table, add `e` or `E` at the start of the hash in the address bar (e.g. `yourpage.html#e` for an empty schedule, or `yourpage.html#e<base64-payload>` if you have saved data).
  2. To hide the schedule (chart-only view), remove the `e`/`E` prefix or use a URL with no prefix.

  The admin char is preserved when saving; adding or removing it in the URL and refreshing updates visibility immediately.

- **Top card** — New card above the schedule and chart cards. Displays the project name as a read-only title (synced from the schedule card's Project Name input). Always visible regardless of admin char. No collapse control.

- **Settings / Advanced split** — The chart card's controls are now organized into two tiers:
  - **Settings** (gear icon in card top bar): Show weekends, Show dependencies, Day/Week/Month view mode, Zoom in/out, Go to today, and an Advanced button (floated right).
  - **Advanced** (toggled by the Advanced button): Accent theme, Row height, Bar height, Bar corner radius, Outline width, Header height, and the Default/Active/Done/Critical/Milestone status color buttons.

- **Text schedule export / import** — New card below the chart that lets you copy or paste the entire schedule as a plain-text block:
  - Fields are separated by the `^` character (no quoting, no escaping) and columns are space-padded for readable monospace alignment. Task names pad to 44 characters, numerical and enum columns pad to their own widths, and the optional `# project:` metadata line carries the project name.
  - Paste a full block into the textarea to replace the entire table (a confirmation dialog guards the replacement). Typing is disabled; the textarea accepts paste only.
  - The right-click **Paste** command works in the text block (read-only lock was dropped in favor of event-level write suppression so context-menu paste still fires).
  - The `^` character is reserved. Attempting to type or paste it into any table / project field pops a warning and the character is stripped on import.
  - Legacy tab-separated blocks still import cleanly; extra whitespace is ignored.

### Changed

- **Chart card header** — Project name title removed from the chart card (now shown in the top card). Settings gear icon moved to the card top bar, to the left of the collapse control, with a smaller icon size (20px).
- **Collapsed card height** — Collapsed cards are now more compact with reduced bottom padding and the collapse icon vertically centered.
- **Chart defaults** — Row height default lowered from 37 to **36**; bar height default lowered from 29 to **26**.
- **Schedule table** — Name column widened (min-width 20rem) for longer task names before horizontal scroll kicks in.

### Refactor

_No user-visible behaviour changes. All removals were confirmed unused before deletion._

**Dead CSS removed**

- `.gantt-card-switches`, `.gantt-card-header`, `.gantt-card-header h1`, `.gantt-card-actions` — the corresponding DOM was removed in a prior pass; only the CSS was left behind.
- `@media (max-width: 599px) .gantt-card-header` and `.gantt-card-collapsed .gantt-card-header` — responsive / collapsed overrides for the now-deleted header element.
- `.color-dot-coral`, `.color-dot-magenta` — palette colours only referenced as `PALETTE_KEY_ALIAS` entries (aliases for `orange` / `rose`); no DOM element ever receives these class names.
- `.field-group--colors`, `.field-group--colors .field` — layout helpers not referenced anywhere in the HTML.

**Dead JavaScript removed**

- `lighten(hex, amount)` — utility defined in the chart script but never called.
- `syncEditForm()` — empty stub (`/* Chart-only: no edit form to sync */`); declaration and its sole call inside `processDragMove` removed.
- `badgeH` variable in `computeLeftWidth` — declared but never read within that function (the live `badgeH` constant in `drawRows` is unaffected).
- Exploratory fallbacks in `getParentId`, `getScheduleData`, and the `target-select` change handler that were added during a suspected linking bug were reverted. The bug turned out to be a user-side mis-link; the simpler 3-line `getParentId` is restored.
- `applyStartEndToRow` reverted to firing flatpickr `onChange` on programmatic `setDate` (no behaviour change observed).

**Consolidated**

- `firstCardTitle` event listeners — three overlapping listeners (two `'input'`, one `'change'`) that each called `syncSecondCardTitle`, updated `document.title`, and persisted to the URL were collapsed into a single `'input'` handler.

---

## v1.4 — Released

### Added

- **Status config dialogs** — Default, Active, Done, Critical, and Milestone each open a modal with a color picker. Active, Done, Critical, and Milestone dialogs also have a "Label" text input for custom display names.
- **Custom status labels** — Pills in the chart and property buttons in the table use custom names. Table buttons show the first letter of the custom label (e.g. "In Progress" → "I").
- **Project name persistence** — Project name is saved in the URL hash and restored on load. Browser tab title shows project name (e.g. "My Project - WFSchedule") for bookmarks.

### Changed

- **Chart defaults** — Week mode and zoomed all the way out (20px per unit).
- **Zoom** — All levels 15% smaller: min 20, max 136, step 10.
- **Header labels** — Unit labels (month-year, week number, date) centered between gridlines. Corner labels (Month-Year, Date/Week Number) have 39px right padding.
- **Name column** — Auto-sizes from widest content; manual setting removed.
- **Color picker in dialogs** — Uses `position: fixed` (M3 pattern) so menus are not clipped; positioned above or below based on viewport space.

### Removed

- **Name column width** slider from settings panel.

---

## v1.3 — Released

### Added

- **Accent themes** — Six accent themes (blue, green, purple, teal, orange, rose) affecting backgrounds, chart colors, and controls. Selector in settings panel.
- **Chart corner labels** — Top-left cell shows "Month - Year" / "Date" (day), "Month - Year" / "Week Number" (week), or "Year" / "Month" (month). Right-aligned.
- **Color picker tooltips** — Single-word color names on hover (no delay) in status and row color pickers.
- **14 status colors** — Palette expanded from 12 to 14: Purple, Indigo, Blue, Cyan, Teal, Green, Lime, Amber, Orange, Red, Rose, Slate, Taupe, Gray. Slate, Taupe, and Gray are unsaturated for clear distinction.

### Changed

- **M3 design system** — Color system uses `--md-sys-color-primary`; M3 type scale; shape tokens; 48px touch targets; responsive layout (compact &lt;600px, medium 600–839px, expanded 840px+); safe-area insets; `100dvh` for mobile.
- **Chart theme** — Chart colors now read from CSS variables (`--wf-chart-*`); dark mode reads from `body`; weekend shading uses `--wf-chart-weekend`.
- **Status color defaults** — Default=gray, Active=green, Done=slate, Critical=red, Milestone=teal.
- **Accent picker** — Moved from top bar into settings panel.
- **Week mode header** — Year shown only when it changes: first month and each January show "Month Year"; other months show only "Month".
- **Card collapse** — Toggle moved to top-right of both cards. When collapsed, all content hidden; only the arrow remains. Compact arrow, theme color, no circular background.
- **Table layout** — Reduced row padding; smaller delete/drag/flag controls; removed add-row-bar border; cards full-width (no max-width).
- **Chart drag/resize** — requestAnimationFrame throttling; persistence deferred until pointer up (`skipPersist`) for smoother interaction.
- **Color pickers** — M3 outline style; circular (not oval); tighter horizontal spacing in settings.
- **Labels** — Unified label style across Project Name, settings sliders, and color pickers.
- **Theme toggle** — Half size; 20% vertical padding; no fill in light mode.
- **Spacing** — Reduced padding between project name and add-row bar; tighter spacing around collapse buttons.

### Fixed

- Chart dark mode (theme read from `body`).
- Accent select dropdown arrow visibility in light and dark modes.
- Undefined `--header-text` CSS variable (replaced with `var(--text)`).

### Refactor

- **Schedule card** — Added `.schedule-card` class; replaced `.card:first-of-type` throughout.
- **Unused CSS removed** — `.nav`, `.nav-title`, `.nav-right`; `--wf-chart-boundary`, `--wf-chart-month-alt`; duplicate `.theme-toggle` rules.

---

## v1.1 — Released

### Added

- Milestone support: `M` flag in the table now renders as a diamond-shaped marker on the chart, using the same color scheme as normal bars.
- Milestone labels are drawn to the right of the diamond for readability.

### Changed

- Bar color logic: bars (and milestones) are colored by flags (Active/Done/Milestone/Critical) instead of the hidden Color column.
- Delay semantics: **Delay 0** now means “same day as the parent’s link date”; non-zero delays offset from that date in the row’s units.
- Default delay for new rows is now **1**, so freshly-added children naturally start one unit after their parent.
- Milestone rows force duration to `0`, end date to equal start date, and delay to `-1` when toggled on.

### Fixed

- Kept link, From, and Delay values stable when toggling Milestone on a linked row, while still re-cascading all progeny when the parent’s dates change.

### Notes

- The Color column is hidden in the table but preserved in the data, so older links and future custom color behavior remain compatible.

---

## v1.0 — Released

Baseline release captured in `Rev 1.0/`.

---

## v1.2 — Released

### Changed

- Chart drag/resize snapping is finer than the visible grid:
  - **Week** view: weekly grid, move/resize in **1-day** steps; bar edges align exactly with day boundaries at any zoom.
  - **Month** view: monthly grid, move/resize in **1-week** steps.
- URL payload is now a compact array format (shorter links, all row fields). **Not backward-compatible with v1.0/v1.1 URLs.**
- Default colors: standard bars **blue**, milestones **orange**. Chart URL persists the status color scheme (Default/Active/Done/Critical/Milestone).

### Fixed

- Milestone diamonds sit on the end-of-day gridline in all views; in week view the diamond center is exactly on that boundary.
- Delay `0` (same-day link) is preserved when saving/restoring from the URL.

