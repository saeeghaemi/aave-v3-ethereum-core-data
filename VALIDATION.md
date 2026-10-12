# Validation

Validation combines prior independent Ethereum checks with release-specific, full-table checks. Passing samples do not imply an RPC census of every row.

## Independent blockchain checks

Evidence collected on 2026-10-11 with fixed seed `20261011`:

| Check | Result |
|---|---:|
| Event receipt/log samples | 64/64 passed |
| Hourly reserve-state samples | 16/16 passed |
| Hourly state field comparisons | 288/288 passed |
| Independently reconstructed concentration samples | 16/16 passed |
| Selected withdrawal-state samples | 12/12 passed |
| Historical balance calls within those samples | 60/60 passed |

## Publication checks

| Dataset | Rows | Duplicate keys | Canonical full-table fingerprint |
|---|---:|---:|---|
| Hourly reserve state | 131,944 | 0 | MATCH |
| Supplier-address concentration | 131,944 | 0 | MATCH |
| Withdrawal liquidity observations | 988,043 | 0 | MATCH |

The two compressed CSV files match the corresponding Parquet row counts, headers, and schema-cast full-table fingerprints. Source hashes were unchanged during export.

### Hourly reserve state

- 129,006 available rows; 2,938 pre-initialization rows.
- Zero duplicate asset-hours.
- Zero active rows with missing Oracle prices.
- Zero `aave_usage_ratio` values outside `[0,1]`.
- Four available zero-denominator rows correctly have null usage ratio.
- No noncanonical utilization field is present.

| Asset | First available hour UTC | Last available hour UTC | Available hours |
|---|---|---|---:|
| DAI | 2023-01-27 07:00 | 2026-10-06 09:00 | 32,355 |
| USDC | 2023-01-27 07:00 | 2026-10-06 09:00 | 32,355 |
| USDT | 2023-02-13 13:00 | 2026-10-06 09:00 | 31,941 |
| WETH | 2023-01-27 07:00 | 2026-10-06 09:00 | 32,355 |

Active coverage is gap-free.

### Supplier-address concentration

- Zero bound or top-1/top-5/top-10 ordering violations.
- Zero reconciliation rows outside the 0.01% tolerance.
- Maximum absolute signed relative reconciliation error: `7.153593151853826e-05`.
- Address-level metrics must not be interpreted as beneficial-owner or entity concentration.

### Withdrawal liquidity observations

Zero duplicate `(block_number, transaction_hash, log_index)` keys.

| Asset | Rows | First event UTC | Last event UTC |
|---|---:|---|---|
| DAI | 27,768 | 2023-01-27 14:24:47 | 2026-10-06 08:02:23 |
| USDC | 330,538 | 2023-01-27 12:14:11 | 2026-10-06 09:50:59 |
| USDT | 263,861 | 2023-02-13 16:53:47 | 2026-10-06 09:55:59 |
| WETH | 365,876 | 2023-01-27 08:22:23 | 2026-10-06 08:56:11 |

Right-censored future states remain null:

| Observation | Missing rows | Complete rows |
|---|---:|---:|
| Prior hour end | 0 | 988,043 |
| 1-hour horizon | 35 | 988,008 |
| 3-hour horizon | 146 | 987,897 |
| 6-hour horizon | 252 | 987,791 |
| 12-hour horizon | 473 | 987,570 |
| 24-hour horizon | 1,063 | 986,980 |
| 48-hour horizon | 2,106 | 985,937 |

Price provenance: 9,613 exact event-block Aave Oracle rows and 978,430 exact hour-end Aave Oracle fallback rows.

## File and scope safety

- Every file is below GitHub's 100,000,000-byte limit.
- No wallet balance/history file is included.
- No shock, recovery, resilience, herd, regression, raw extraction, database, credential, RPC configuration, or machine-specific path is included.
- SHA-256 hashes in `checksums.sha256` cover every published file except the manifest itself.

## Remaining limitations

- Independent RPC checking is stratified sample evidence, not a census.
- Most withdrawal prices use a labeled hour-end Oracle fallback rather than an event-block price.
- Concentration is address-level, not entity-level.
- Horizon states are exact hour-end observations, not exact event-time-plus-horizon calls.
- No immediate pre-withdrawal liquidity is available; block-start liquidity is not equivalent.
- Future horizons near 2026-10-06 are right-censored.
- The release is a fixed snapshot and is not current beyond its documented endpoint.
