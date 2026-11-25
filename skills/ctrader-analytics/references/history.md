# Trade history

## Time windows and completeness

`get_deals` and `get_order_history` require both `fromTimestamp` and `toTimestamp`. Send strings containing a valid ISO 8601 date-time with `Z` or an explicit offset, or integer epoch milliseconds as a string. Resolve relative dates in the user's time zone, then send explicit boundaries, preferably in UTC. Date-only values, date-times without a zone, epoch seconds and microseconds are invalid. The supported timestamp bounds are `1000000000000` through `4102444800000` milliseconds, inclusive (2001-09-09T01:46:40Z through 2100-01-01T00:00:00Z). Keep the start before the end and each request within 30 days. Correct a rejected value from the intended date and unit; do not guess a replacement interval.

`get_deals.maxRows` defaults to 50 and is clamped to 1000. The returned `hasMore` flag means the response is incomplete. Narrow or subdivide the time window, overlapping boundary timestamps when necessary and deduplicating by `dealId` or `orderId`. Do not assume an undocumented ordering, advance by one millisecond and lose equal-timestamp events, or invent a cursor/offset argument.

If a full single-timestamp group cannot be retrieved within the available limits, state that coverage is incomplete. `get_pending_orders` has no history-window or paging arguments; if it reports truncation, do not try to page it using parameters belonging to the history tools.

## Executions and realised results

Orders record instructions and lifecycle events. Deals record executions, including partial fills. Do not count every order as a trade or infer filled volume from requested volume. Use `filledVolume` and the returned `dealStatus`.

`executionPrice`, order prices, position entry/close prices and protection prices are already display values. They are not raw quote integers.

A closing deal can include `closing: true` and `closePositionDetail`. Use that close summary's `grossProfit`, `netProfit`, `commission`, `swap`, `closedVolume` and `moneyDigits` where present. Convert money once and preserve charge signs. Do not add the same closing commission again from the deal's top-level field.

`get_position_details` can return a closed position, related orders/deals, close price/time and aggregate realised fields. A position with some realised profit can still have open volume after a partial close. Separate the realised part from its remaining unrealised result.

Check `positionStatus` and execution evidence before classifying a position. `POSITION_STATUS_CREATED` has not filled; `POSITION_STATUS_ERROR` describes an opening failure. A `POSITION_STATUS_CLOSED` record with no `openTimestamp` and an explicitly empty `deals` array never opened. Its cancelled, rejected or expired order is not a completed trade and has no trade P&L. Do not assign that classification from one missing field or an omitted `deals` array.

Position-level `commission` and `swap` cover the open part; `closedCommission` and `closedSwap` cover the closed part. Add the corresponding components only when returned, scale with the position's `moneyDigits`, and preserve their signs. Use the close summary or the position totals for a given result, without adding both representations of the same profit or fees. Do not subtract charges again from returned `netProfit`. Missing profit or fee fields remain unavailable, even if prices and execution volumes are present.

## Performance summaries

Define the population before calculating a win rate or average return: closing deals, fully closed positions, or another explicit grouping. Multiple closing deals can belong to one position; group by `positionId` when the metric is per position. Exclude unfilled and rejected executions from completed-trade counts.

For an interval result, prefer the closing deals whose execution timestamps fall inside that interval and verify coverage. Position-detail realised totals accumulate across the position's closes and may include activity before the requested start. Do not attribute the whole total to the interval merely because a related deal or final close falls inside it. Use a position total only when the contributing closes belong to the selected population and interval; otherwise report the aggregate separately and state that interval P&L is unavailable. Do not add that aggregate to its closing deals again.

A cash P&L total is not a percentage return. Do not infer an equity curve, drawdown, time-weighted return or money-weighted return from a current balance and a partial deal list. Report unavailable inputs and incomplete history. No returned deals means no matching records for that verified interval, not proof that the account has never traded.

Recent history can lag other account views. For discrepancies, use position details and bounded read-only rechecks; report unresolved differences. Never create or close a trade to produce sample history for an analysis.
