# Account data

## Account snapshot

1. Read `get_balance` for the connected account.
2. Resolve its `depositAssetId` with `get_assets`. Divide balance, equity and free margin by `10^moneyDigits` from that balance response.
3. Read `get_positions` for open positions. Its response can also include orders. Use `get_pending_orders` when the request requires the pending-order view; do not count the same order twice.
4. Resolve unfamiliar symbol IDs with `get_symbols` when names are needed. Reuse an already retrieved mapping for the same connection.
5. Present balance, equity, free margin, positions and pending orders separately. Independent reads are snapshots at different times, not an atomic account statement.

## Exposure and protections

Report position direction and base-asset units (`volume / 100`). Do not add units across unrelated assets or present that sum as account-currency exposure. Account-currency notional needs the corresponding quotes and conversion rates; state the time and assumptions if calculating it.

Position `entryPrice`, `stopLoss`, `takeProfit`, `closePrice` and prices inside `advancedTakeProfit` use display prices. Position protection levels describe existing settings. Reading them does not authorize changing them.

An absent stop loss or take profit means that no value was returned. Describe the data rather than automatically adding a protection. A pending order is not a filled position; a protective order associated with a position is not necessarily a separately requested trade.

`get_positions` is the open-position view: membership in its `positions` array establishes that the position was open at that read, even when the row has no `positionStatus`. A separate position-detail response without status does not establish whether it is open or closed.

When `positionStatus` is present, distinguish `POSITION_STATUS_CREATED` (the pending order has not filled), `POSITION_STATUS_OPEN`, `POSITION_STATUS_CLOSED` and `POSITION_STATUS_ERROR` (the opening order failed). A created position has no filled exposure to close. Do not count an explicitly created, closed or errored record as an open trade.

`POSITION_STATUS_CLOSED` alone does not prove a completed trade. If the status is `POSITION_STATUS_CLOSED` and position details contain no `openTimestamp` and an explicitly empty `deals` array, the position never opened: its order was cancelled, rejected or expired before a fill. Report that state without assigning trade P&L. If the response omits the evidence needed to distinguish these states, report the uncertainty.

The symbol catalogue supplies identifiers, names, enabled state, asset IDs, description, `digits` and `pipPosition`. It does not supply authoritative lot size or minimum/maximum/step volume. Do not use a fixed forex lot size for metals, indices or crypto, and do not infer missing limits by placing orders.

## Monetary values

Read amounts in the account's deposit currency. If an enclosing position detail or closing-deal summary includes `moneyDigits`, use that value for its money fields. Otherwise use the connected account's `get_balance` value. Never scale `positionId`, `symbolId`, `balanceVersion`, timestamps or volume as money.

Use server-returned realised and unrealised results where provided. Do not reconvert a money field from a symbol's quote currency when it is already expressed in deposit currency. Preserve the returned signs of commission and swap; do not subtract a negative charge twice.

On a position detail, `commission` and `swap` cover the still-open part; `closedCommission` and `closedSwap` cover the closed part. When the response supplies both components, full position commission is `commission + closedCommission`, and full swap is `swap + closedSwap`, scaled with that position's `moneyDigits`. A zero open-part commission on a closed position does not imply zero total commission. Report absent components as missing rather than replacing them with zero. Use returned `netProfit` directly when available; do not deduct its included fees again or add position totals to the same deals' charges.

An open position can have realised `grossProfit`, `netProfit` and closed-fee totals after a partial close. Keep those totals separate from the remaining unrealised result. These position totals can span more than the requested reporting interval; use the [history rules](history.md) before including them in an interval P&L calculation.

If used margin is not supplied, `equity - freeMargin` may be presented only as a derived value from the same balance response. Do not invent a stop-out threshold, hedge mode, leverage tier or broker-specific risk limit from this calculation.

## Find an order by client ID

If the connected schema exposes `get_order_by_client_order_id`, use it for an order lookup when the user or returned data supplies the actual `clientOrderId`. The parameter is a string of 1 to 50 characters. A `clientOrderId` differs from an `orderId`, label or comment; preserve the exact value, including a returned server-generated `CTGW-...` value.

Report `ORDER_NOT_FOUND` as no matching order returned for this connection. This lookup does not authorize creating or resending an order. Keep order creation and retry workflows outside this read-only skill.
