# Changelog

## v1.0 — 8 Oct 2026

First public release of the anonymised sample.

### Added
- `PPC_Order_Planner_SAMPLE.xlsx`: PO scheduling on a `WORKDAY.INTL` calendar with style chaining, manual start overrides, buffer days and OTIF (projected and actual).
- `PPC_Machine_Capacity_Planner_SAMPLE.xlsx`: 26-line × 105-day line plan, 42 style recipes and daily demand for 26 machine groups against 920 installed machines, with day and month dashboards.
- Docs: [order planner](docs/order-planner.md), [capacity planner](docs/capacity-planner.md) and the [review and fix log](docs/fix-log.md).
- README banner and screenshots.

### Review fixes (before publication)
Nine review findings were addressed in the working files and carried into the samples, including the overwritten Start Date chaining formula (163 of 191 rows), OTIF measured on a projected date, 18 silently unscheduled open orders and month views hard-coded to Aug–Nov 2026. Each fix and its check is in [`docs/fix-log.md`](docs/fix-log.md).

### Changed
- 10 Oct 2026: dashboard, MD view and order planner screenshots replaced with renders from the sample workbooks' calculated values.

## Repository maintenance

- 17 Oct 2026: added `.gitattributes` (workbooks and images binary, LF for docs).
- 18 Oct 2026: added this changelog.
