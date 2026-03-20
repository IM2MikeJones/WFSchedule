# Release notes — WFSchedule

This file tracks changes **going forward**.

- **How you use it**: when you decide a release is “done”, you create a new `Rev X.Y/` folder, move `index.html` + docs (including this file) into it, then tell me to start a fresh **Unreleased** section for the next version.

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

