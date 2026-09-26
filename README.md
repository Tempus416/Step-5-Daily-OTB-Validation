# Step-5-Daily-OTB-Validation
Validates the Daily OTB pipeline's output against the source system's own totals and hand-verified real cases — total-row reconciliation, zero-base STLY variance, negative pickup, and unmapped rate code flow-through — across two hotel properties at full production scale.

# Step 5 Daily OTB Validation

Validates the Daily OTB ETL pipeline (Steps 1-4) against the source system's own data, since this project's dataset is itself the training/example data — there's no independent external report covering the same period to check against. Validation here means proving the pipeline is internally correct and faithful to the original 365-Day Pickup report's logic, not matching an outside source.

## What This Validates

| Check | Method | Result (43 snapshots, 2 properties) |
|---|---|---|
| **Total-row reconciliation** | Compares our cleaned data's summed Rooms Sold/Revenue against the source file's own "Total" row for each snapshot | PASSED — Rooms Sold exact match; Revenue within ~0.001% (industry-standard tolerance: ±3%) |
| **Zero-base STLY variance** | Confirms 0-this-year vs. 0-last-year correctly produces 0.0%, not NaN/inf | PASSED — 34,557 real rows checked |
| **Negative pickup / historical immutability** | Hand-verified a real cancellation case, confirmed past stay dates never change across later snapshots | PASSED — exact match across 42 consecutive snapshot transitions |
| **Unmapped rate code flow-through** | Confirms a rate code not yet in the reference mapping table still flows through pickup/variance rather than silently dropping | PASSED — 206 rows, correctly flagged, never lost |

## A Genuine Finding, Not Just a Pass/Fail

The Total-row reconciliation surfaced a small, **consistent** dollar gap (~$100-120/day) between our computed revenue and the source's stated total, on both properties independently. Critically, this gap does not scale with revenue volume (it stays flat even as daily revenue grows 9-38% over the test period) — pointing to a specific unattributed daily line item (e.g., a flat resort fee) rather than generic floating-point rounding. Comfortably within tolerance, but flagged for confirmation rather than dismissed, since a stable pattern that shows up independently in two separate datasets is worth explaining, not just tolerating.

## Design Note

This notebook is fully standalone — it re-runs the entire pipeline (ingestion, cleaning, business logic) itself rather than depending on Steps 1-4 having been run first in the same session. This means Steps 1-4's functions currently exist in two places (their own notebooks and duplicated here), a deliberate tradeoff for now, to be resolved in Step 6 when everything is consolidated into shared, importable modules.

## Requirements

- Python 3.x, `pandas`, `numpy`, `openpyxl`
- Reference/dimension workbooks and standardized daily OTB files for each property (see Steps 1-4 repos)
