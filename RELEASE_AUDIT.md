# Release audit

> This audit describes the immutable tagged v1.0.0 release. Validated Excel convenience copies added later on the `main` branch do not alter that tag or its canonical datasets.

**Release:** v1.0.0  
**Audit date:** 2026-10-13  
**Author:** Mohammad Saee Ghaemi

## Publication decision

| Dataset group | Status | Rows | Basis |
|---|---|---:|---|
| Hourly reserve state | READY | 131,944 | Exact mapped blocks; canonical usage ratio; no duplicates, invalid active prices, or invalid ratios. |
| Supplier-address concentration | READY | 131,944 | Reconciliation and independent sampled reconstruction passed; address-level interpretation documented. |
| Withdrawal liquidity observations | READY | 988,043 | Full withdrawal universe; no duplicate event keys; timing, price provenance, and right-censoring explicit. |

Release-level status: **READY** for publication within the documented scope.

## Scope gate

Included data directories are exactly:

- `data/reserve_hour`
- `data/depositor_concentration`
- `data/liquidity_observations`

Excluded material is absent: wallet balance intervals or histories, shock labels, recovery/resilience/herd outputs, regressions, DuckDB, raw extraction chunks, credentials, and private configuration.

## Integrity gate

- Row counts and canonical full-table fingerprints match.
- Duplicate counts are zero.
- CSV/Parquet equivalence checks pass for both dual-format tables.
- No data file exceeds GitHub's 100,000,000-byte file limit.
- Binary data files are stored through Git LFS without changing their contents; the manifest hashes the restored file bytes.
- `checksums.sha256` is recalculated from the exact publication tree.
- The source release remains separate and unchanged.

## Licensing gate

The repository contains public blockchain-derived data and original documentation, but no copied Aave source code. Data and documentation are licensed under CC BY 4.0. No software is included, so a separate MIT code license is unnecessary.
