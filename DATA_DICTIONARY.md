# Data dictionary

## Common conventions

| Convention | Definition |
|---|---|
| Network | Ethereum Mainnet, `chain_id = 1`. |
| Market | Aave V3 Ethereum Core, stored as `AaveV3Ethereum`. |
| Assets | USDC, USDT, DAI, WETH. |
| UTC | All timestamps carry UTC metadata in Parquet. |
| Hour label | `hour_utc = t` labels `[t, t + 1 hour)`. State is taken at the last canonical block at or before `t + 59:59`. |
| Native unit | Human-readable token amount after division by `10^token_decimals`. |
| Raw integer | Exact Solidity integer represented as a decimal string when it may exceed Arrow's signed integer range. |
| Ray | Aave fixed-point value scaled by `10^27`, retained as an exact decimal string. |
| Missing value | Unavailable or right-censored observation; never silently zero-filled. |

## `data/reserve_hour/reserve_hour.parquet`

One row per asset and UTC hour. The compressed CSV contains the same 131,944 rows and columns.

| Variable | Type/unit | Definition |
|---|---|---|
| `hour_utc` | UTC timestamp | Start label of the hour whose end state is observed. |
| `block_number` | integer | Greatest canonical Ethereum block at or before the hour-end second. |
| `chain`, `chain_id`, `market`, `asset` | identifiers | Network, chain ID, Aave market, and reserve symbol. |
| `reserve_address` | address | Underlying token contract. |
| `atoken_address` | address | Aave interest-bearing token contract. |
| `variable_debt_token_address` | address | Variable-debt token contract. |
| `stable_debt_token_address` | address | Stable-debt token contract returned by Pool reserve data. |
| `token_decimals` | integer | Underlying token decimals. |
| `state_available` | boolean | Whether the historical reserve/token state is available. |
| `availability_reason` | string/null | Reason for an unavailable state. |
| `state_source` | string | Historical-call provenance label. |
| `total_supplied_native` | native token | aToken total supply after decimal adjustment. |
| `total_variable_borrowed_native` | native token | Variable-debt-token total supply after decimal adjustment. |
| `total_stable_borrowed_native` | native token | Stable-debt-token total supply after decimal adjustment. |
| `total_debt_native` | native token | Variable plus stable debt. |
| `available_liquidity_native` | native token | Underlying token `balanceOf(aToken)` after decimal adjustment. |
| `aave_usage_ratio` | fraction/null | `total_debt_native / (available_liquidity_native + total_debt_native)`. Null for unavailable or zero-denominator states. |
| `price_usd` | USD/token | Aave Oracle price at the hour-end block. |
| `total_supplied_usd`, `total_debt_usd`, `available_liquidity_usd` | USD | Native values multiplied by the matching Oracle price. |
| `liquidity_index`, `variable_borrow_index` | ray string | Exact Aave reserve indices. |
| `liquidity_rate_ray`, `variable_borrow_rate_ray`, `stable_borrow_rate_ray` | ray string | Exact annualized rate fields in ray units; no APY conversion is included. |
| `ltv`, `liquidation_threshold` | fraction | Risk parameters decoded from the reserve configuration bitmap. |
| `liquidation_bonus` | multiplier | Configuration value divided by 10,000. |
| `is_active`, `is_frozen`, `borrowing_enabled`, `stable_borrowing_enabled`, `is_paused` | boolean | Reserve configuration flags. |
| `borrowable_in_isolation`, `siloed_borrowing`, `flashloan_enabled` | boolean | Reserve configuration flags. |
| `reserve_factor` | fraction | Reserve factor decoded from configuration. |
| `borrow_cap`, `supply_cap` | whole native tokens | Cap values; zero follows Aave's uncapped convention. |
| `liquidation_protocol_fee` | fraction | Liquidation protocol fee decoded from configuration. |
| `debt_ceiling` | raw configuration integer | Isolation-mode debt ceiling; not silently rescaled. |
| `virtual_accounting_enabled` | boolean | Virtual-accounting configuration flag. |

## `data/depositor_concentration/depositor_concentration.parquet`

One row per asset/hour. “Supplier address” means an address with a positive reconstructed aToken balance; it does not mean a unique beneficial owner.

| Variable | Type/unit | Definition |
|---|---|---|
| `hour_utc`, `asset`, `block_number` | identifiers | Asset-hour and matching reserve-state block. |
| `state_available` | boolean | Whether the underlying reserve state is available. |
| `concentration_status` | string | Reconstruction availability/status. |
| `reconciliation_status` | string | `pass`, `pre_initialization`, `zero_supply`, or `outside_tolerance`; no published row is outside tolerance. |
| `reconciliation_tolerance` | fraction | Maximum permitted absolute signed relative error, 0.0001. |
| `signed_reconciliation_error_native` | native token | Signed reconstructed total minus reserve aToken total supply. |
| `signed_relative_error` | fraction/null | Signed reconciliation error divided by reserve total supply. |
| `total_supplied_native` | native token | Reserve-level aToken total supply. |
| `price_usd` | USD/token | Aave Oracle hour-end price. |
| `number_active_supplier_addresses` | integer | Count of positive reconstructed address balances. |
| `largest_supplier_address_balance_native` | native token | Largest positive reconstructed address balance. |
| `top_5_supplier_address_balance_native` | native token | Sum of the five largest positive address balances. |
| `top_10_supplier_address_balance_native` | native token | Sum of the ten largest positive address balances. |
| `top_1_supplier_address_share` | fraction | Largest address balance divided by reserve total supply. |
| `top_5_supplier_address_share` | fraction | Top-five address balance divided by reserve total supply. |
| `top_10_supplier_address_share` | fraction | Top-ten address balance divided by reserve total supply. |
| `supplier_address_hhi` | fraction [0,1] | Sum of squared shares using the positive reconstructed address-balance total as denominator. |

## `data/liquidity_observations/liquidity_observations_<asset>.parquet`

One row for every in-scope Aave Pool `Withdraw` event. Combined row count: 988,043.

| Variable or pattern | Type/unit | Definition |
|---|---|---|
| `withdrawal_timestamp_utc` | UTC timestamp | Exact block timestamp of the event. |
| `event_hour_utc` | UTC timestamp | Hour bucket containing the event. |
| `block_number`, `transaction_hash`, `log_index` | identifiers | Ethereum event identity. |
| `chain_id`, `market` | identifiers | Ethereum Mainnet and Aave market. |
| `pool_contract_address`, `reserve_address` | address | Emitting Pool and underlying reserve contracts. |
| `asset` | string | Reserve symbol. |
| `amount_raw` | decimal string | Exact emitted withdrawal amount. |
| `token_decimals` | integer | Underlying token decimals. |
| `withdrawal_amount_native` | native token | Decimal-adjusted amount. |
| `price_usd` | USD/token | Labeled Aave Oracle price used for the event. |
| `price_match_method` | string | `exact_event_block_aave_oracle` or `fallback_hour_end_aave_oracle`. |
| `withdrawal_amount_usd` | USD | Native amount multiplied by the labeled price. |
| `event_source`, `event_source_query_group` | string | RPC/event extraction provenance. |
| `state_observation_hour_prior_hour_end_utc` | UTC timestamp | Hour immediately before the event hour; not an immediate pre-event state. |
| `state_observation_hour_<h>h_hour_end_utc` | UTC timestamp/null | `floor(withdrawal timestamp + h hours)` for `h ∈ {1,3,6,12,24,48}`; null when right-censored. |
| `state_block_number_<suffix>` | integer/null | Exact end-of-hour block for the corresponding state. |
| `available_liquidity_<suffix>_native` | native token/null | Underlying `balanceOf(aToken)` at that hour-end block. |
| `total_debt_<suffix>_native` | native token/null | Variable plus stable debt at that hour-end block. |
| `aave_usage_ratio_<suffix>` | fraction/null | Debt divided by available liquidity plus debt at the corresponding state. |

`<suffix>` is `prior_hour_end`, `1h_hour_end`, `3h_hour_end`, `6h_hour_end`, `12h_hour_end`, `24h_hour_end`, or `48h_hour_end`.
