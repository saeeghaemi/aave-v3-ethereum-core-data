# Methodology

## Scope

- Ethereum Mainnet (`chain_id = 1`)
- Aave V3 Ethereum Core (`AaveV3Ethereum`)
- USDC, USDT, DAI, WETH
- Requested panel start: 2023-01-01 00:00 UTC
- Release end: 2026-10-06 09:00 UTC, the latest fully confirmed hour in the build
- Confirmation rule: 64 blocks behind the safe head

The requested start predates reserve-token initialization. Those hours are retained with unavailable state and null values, not interpreted as zero balances.

## Sources

The source pipeline uses Ethereum JSON-RPC and official Aave contracts.

| Component | Address |
|---|---|
| Pool | `0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2` |
| PoolAddressesProvider | `0x2f39d218133AFaB8F2B819B1066c7E434Ad94E9e` |
| Aave Oracle | `0x54586bE62E3c3580375aE3723C145253060Ca0C2` |

Retrieval uses Pool `Withdraw` logs; historical Pool reserve/configuration calls; historical aToken and debt-token total supplies; underlying `balanceOf(aToken)`; historical Aave Oracle prices; and canonical block timestamps. No subgraph, Dune table, centralized-exchange price, or interpolated blockchain state is mixed into the release.

Official reference interfaces and addresses:

- <https://github.com/aave-dao/aave-v3-origin/blob/main/src/contracts/interfaces/IPool.sol>
- <https://github.com/aave-dao/aave-v3-core/blob/master/contracts/interfaces/IAToken.sol>
- <https://github.com/aave-dao/aave-v3-core/blob/master/contracts/protocol/tokenization/base/ScaledBalanceTokenBase.sol>
- <https://github.com/aave-dao/aave-address-book/blob/main/src/AaveV3Ethereum.sol>

## Exact hourly state

For an hour labeled `t`, the observation block is the greatest canonical block whose timestamp is at or before `t + 1 hour − 1 second`.

```text
total_supplied_native = aToken.totalSupply / 10^token_decimals
total_variable_borrowed_native = variableDebt.totalSupply / 10^token_decimals
total_stable_borrowed_native = stableDebt.totalSupply / 10^token_decimals
total_debt_native = total_variable_borrowed_native + total_stable_borrowed_native
available_liquidity_native = underlying.balanceOf(aToken) / 10^token_decimals
aave_usage_ratio = total_debt_native
                   / (available_liquidity_native + total_debt_native)
```

The source analytical debt/aToken-supply ratio is not exported. A zero denominator produces null. USD stocks use the Aave Oracle price from the same hour-end block.

## Supplier-address concentration

Concentration is derived from complete aToken `Mint`, `Burn`, and `BalanceTransfer` streams. Economic scaled balances are reconstructed with Aave ray arithmetic, valued with each hour's liquidity index, and reconciled to reserve-level aToken total supply within an absolute relative tolerance of 0.01%.

Top-address shares divide the top address-balance sums by reserve-level aToken supply. HHI is the sum of squared positive address shares using the reconstructed positive-balance total as denominator.

The underlying address-balance histories are not part of this public release. Published concentration measures describe addresses, not persons or independent entities. Exchanges, custodians, bridges, smart-contract wallets, and common-control relationships are not comprehensively classified.

## Withdrawal liquidity observations

The table starts from all 988,043 in-scope Pool `Withdraw` events. For each event it joins:

1. the end-of-hour state immediately before the event's hour; and
2. the hour-end state labeled `floor(event timestamp + h)` for `h = 1, 3, 6, 12, 24, 48` hours.

These are exact mapped hour-end blockchain observations, not interpolations. Because the exact target instant usually falls inside the selected hour, a horizon state can be observed up to almost one hour after that instant. Events near the release boundary are right-censored and retain null future states.

No immediate pre-withdrawal field is published. A block-`b−1` balance is block-start liquidity, not necessarily liquidity immediately before an event log in block `b`; the source project's block-start/end observations also covered only a pre-screened subset. The release therefore excludes those values and all shock or recovery classifications.

## Price timing

Withdrawal prices retain a provenance label:

- `exact_event_block_aave_oracle`: Aave Oracle price at the event block.
- `fallback_hour_end_aave_oracle`: exact Aave Oracle price at the event hour's end.

No price is interpolated. The fallback must not be treated as an event-block observation.

## Observed and reconstructed variables

| Category | Examples | Interpretation |
|---|---|---|
| Direct blockchain observation | Block, event timestamp, emitted amount, token supply, balance, index, rate, configuration, Oracle price | Returned by an Ethereum log or historical contract call at a specified block. |
| Deterministic transformation | Native amounts, USD stocks, canonical usage ratio | Arithmetic applied to observed fields. |
| Reconstructed aggregate | Supplier-address concentration | Deterministic aToken replay, validated by reserve-level reconciliation. |
| Excluded research construct | Shock, recovery, resilience, herd, causal outputs | Not part of v1.0.0. |

## Licensing note

This repository contains no copied Aave source code. The dataset consists of public blockchain observations and original transformations/documentation. Aave V3 source-code licensing therefore does not govern the repository contents; the dataset and documentation are released under CC BY 4.0. This is a scope assessment, not legal advice, and no Aave trademark rights or endorsement are claimed.
