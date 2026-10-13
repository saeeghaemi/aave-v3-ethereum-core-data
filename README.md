# Aave V3 Ethereum Core Dataset

Research-grade hourly reserve states, supplier-address concentration measures, and withdrawal liquidity observations for Aave V3 Ethereum Core on Ethereum Mainnet.

**Version:** 1.0.0  
**Author:** Mohammad Saee Ghaemi  
**Coverage:** 2023-01-01 00:00 UTC through 2026-10-06 09:00 UTC  
**Assets:** USDC, USDT, DAI, WETH  
**License:** [CC BY 4.0](LICENSE)

## Published datasets

| Dataset | Rows | Format | Description |
|---|---:|---|---|
| Hourly reserve state | 131,944 | Parquet, CSV.gz, Excel | Exact end-of-hour reserve, token, Oracle, rate, index, and configuration observations. |
| Supplier-address concentration | 131,944 | Parquet, CSV.gz, Excel | Top-1/top-5/top-10 address shares, address HHI, and reconstruction reconciliation controls. |
| Withdrawal liquidity observations | 988,043 | Parquet by asset, Excel | Every in-scope Pool `Withdraw` event joined to prior-hour and 1/3/6/12/24/48-hour-end liquidity states. |

Parquet is the canonical format. The two compact hourly tables also have compressed CSV copies.

## Download Excel Datasets

- [Hourly reserve state (.xlsx)](https://media.githubusercontent.com/media/saeeghaemi/aave-v3-ethereum-core-data/main/excel/aave_hourly_reserve_state.xlsx)
- [Supplier-address concentration (.xlsx)](https://media.githubusercontent.com/media/saeeghaemi/aave-v3-ethereum-core-data/main/excel/aave_supplier_concentration.xlsx)
- [Withdrawal liquidity observations (.xlsx)](https://media.githubusercontent.com/media/saeeghaemi/aave-v3-ethereum-core-data/main/excel/aave_withdrawal_liquidity.xlsx)

These Excel workbooks are convenient copies of the validated public datasets. Parquet remains the canonical and preferred format for precise computational research.

Excel worksheets are limited to 1,048,576 rows. The withdrawal workbook contains 988,043 data rows plus one header row and therefore fits in a single worksheet. High-precision numeric fields are stored as text to prevent Excel's 15-significant-digit precision limit from changing the published values. Addresses, transaction hashes, raw token amounts, and ray values are also stored as text. UTC timestamps are stored as Excel date-time cells. Analysts should import text-preserved numeric fields explicitly when calculations are required.

The binary data files are stored with [Git LFS](https://git-lfs.com/). Install Git LFS before cloning, or download individual files from GitHub. A normal Git LFS checkout restores the exact bytes recorded in `checksums.sha256`.

## Directory layout

```text
data/
├── reserve_hour/
│   ├── reserve_hour.parquet
│   └── reserve_hour.csv.gz
├── depositor_concentration/
│   ├── depositor_concentration.parquet
│   └── depositor_concentration.csv.gz
└── liquidity_observations/
    ├── liquidity_observations_dai.parquet
    ├── liquidity_observations_usdc.parquet
    ├── liquidity_observations_usdt.parquet
    └── liquidity_observations_weth.parquet
excel/
├── aave_hourly_reserve_state.xlsx
├── aave_supplier_concentration.xlsx
└── aave_withdrawal_liquidity.xlsx
```

## Time, units, and missing values

- Timestamps are timezone-aware UTC.
- An hour labeled `t` represents `[t, t + 1 hour)`. Its state is observed at the greatest canonical Ethereum block timestamp at or before `t + 59:59`.
- USDC and USDT use 6 decimals; DAI and WETH use 18.
- Large Solidity integers and ray values are stored as decimal strings where needed to preserve precision.
- Missing observations remain null. Pre-initialization hours are unavailable, not zero.
- `aave_usage_ratio` is debt divided by available liquidity plus debt. The noncanonical debt/aToken-supply ratio is not published.

## Quick start

```python
import pandas as pd

reserve = pd.read_parquet("data/reserve_hour/reserve_hour.parquet")
concentration = pd.read_parquet(
    "data/depositor_concentration/depositor_concentration.parquet"
)
withdrawals = pd.read_parquet("data/liquidity_observations/*.parquet")
```

## Scope boundaries

This repository intentionally excludes individual wallet balance histories, wallet interval files, shock classifications, recovery or resilience metrics, herd-behavior indicators, regressions, raw extraction chunks, the project DuckDB database, and private configuration.

Concentration is measured across blockchain addresses, not verified people or economic entities. Withdrawal horizon fields are end-of-hour observations, not exact event-time-plus-horizon calls. The repository contains no immediate pre-withdrawal liquidity field and no causal analysis.

See [DATA_DICTIONARY.md](DATA_DICTIONARY.md), [METHODOLOGY.md](METHODOLOGY.md), and [VALIDATION.md](VALIDATION.md) before analysis. Verify downloads with [checksums.sha256](checksums.sha256).

## Attribution

When reusing the dataset or documentation, cite the author and repository, link to CC BY 4.0, and indicate changes. A suggested attribution is:

> Aave V3 Ethereum Core Dataset v1.0.0, Mohammad Saee Ghaemi, CC BY 4.0.

“Aave” and related marks belong to their respective owners. This independent research dataset is not affiliated with or endorsed by Aave Labs, Aave DAO, or the Aave protocol contributors.
