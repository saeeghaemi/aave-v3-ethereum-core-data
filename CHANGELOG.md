# Changelog

## Main branch — 2026-10-13

- Added validated Excel convenience copies for the three published dataset groups under `excel/`.
- Added direct workbook download links and documented Excel row and numeric-precision limits.
- Preserved the tagged v1.0.0 release and all canonical dataset files unchanged.

## v1.0.0 — 2026-10-13

Initial public release.

### Included

- Hourly reserve-state data with exact end-of-hour blocks, token stocks, Oracle prices, indices, rate rays, and configuration fields.
- Supplier-address concentration data with top-address shares, HHI, and reconstruction reconciliation controls.
- Asset-partitioned liquidity observations for all in-scope withdrawal events.
- Documentation, validation evidence, CC BY 4.0 notice, and SHA-256 manifest.

### Publication corrections

- Uses canonical Aave usage ratio: debt divided by available liquidity plus debt.
- Uses supplier-address language rather than beneficial-owner language.
- Preserves unavailable and right-censored observations as null.
- Clearly labels event-block versus hour-end fallback Oracle prices.

### Excluded

- Individual wallet balances and histories.
- Shock classifications and pre-screened shock subsets.
- Block-start/end fields that could be misread as immediate pre/post-event liquidity.
- Recovery, resilience, clustering, herd, regression, and causal outputs.
- Raw extraction chunks, DuckDB, credentials, and private configuration.

The canonical project datasets were not modified.
