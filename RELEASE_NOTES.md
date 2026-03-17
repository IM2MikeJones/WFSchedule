# Release notes — WFSchedule

This file tracks changes **going forward**.

- **How you use it**: when you decide a release is “done”, you create a new `Rev X.Y/` folder, move `index.html` + docs (including this file) into it, then tell me to start a fresh **Unreleased** section for the next version.

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

## v1.2 — Unreleased

Placeholder for future changes after `Rev 1.1/` is created.

