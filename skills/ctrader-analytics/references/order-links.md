# Order and position links

Use the link tools exposed by the current connection. They return a URL for the user to open in cTrader and perform no trading action. Generate a link only for the user's requested operation and target. Do not turn an analytics question into a trade proposal or call a trading tool when a link tool is unavailable.

## New order

`create_order_link` builds a URL to a prefilled new-order form in cTrader Web or Mobile. It does not place an order. Use it for a new order with parameters chosen by the user. To close or change an existing position or order, use the corresponding link workflow below. An opposite-side new order can open a second position on a hedging account; do not use it as a substitute for closing.

Pass exactly one verified symbol name or ID. Follow the current schema: `orderType` uses lowercase `market`, `limit`, `stop` or `stop-limit`, and `direction` uses lowercase `buy` or `sell`. Non-market links require an entry `price`; a market link has no entry price.

Prices and stop-loss/take-profit values are absolute display prices. Respect the symbol's `digits`. For a pending buy, the stop loss must be below entry and the take profit above it; the inequalities reverse for a sell. Ask about missing user choices rather than inventing them.

`range` uses **pips**, not price units or points. A `stop-limit` link requires a positive `range`: the limit lies that many pips beyond the stop price in the trade direction. A `market` link can optionally include it as the permitted market range. Do not send `range` for `limit` or `stop`. Obtain the user's range rather than selecting one. If the connected schema lacks this parameter, explain that it cannot prepare the fully specified stop-limit link; do not append `r` yourself. The server rejects values that serialize in scientific notation.

Check whether the connected tool schema exposes `volume` before promising a prefilled size. A schema without that parameter cannot include the requested size in the link. Do not send an undeclared argument or append a volume parameter to the returned URL yourself. In that case, return the link with an explicit instruction to enter the desired size manually in cTrader, and state that the form may otherwise use its own default. If the user requires a link with the size already filled in, explain that the connected tool does not support that requirement.

When the connected schema exposes `volume` and the user supplied a size that can be converted from verified data, pass it in the documented unit. For hundredths of a base-asset unit, a user-requested 1000 base units means `volume: 100000`. A request in lots additionally requires verified broker contract size. Do not assume a forex contract size or send a test trade to discover it. If size is omitted or cannot be resolved, explain that cTrader may use its own default and the user must set the intended size before placing the order. Preserve the server's size notice in either case.

Optional `expiry` is an integer in epoch milliseconds, unlike string timestamps used by history tools, and must stay within `1000000000000` through `4102444800000`. `comment` accepts Unicode text with a limit of 100 UTF-16 code units, equivalent to JavaScript `string.length`; an emoji can use several units. Trim on complete characters and keep within that limit. The server rejects a split surrogate pair. Avoid sensitive account information in comments.

## Existing position or pending order

| User request | Link workflow |
| --- | --- |
| Close a position or change its SL/TP | Identify the current position with `get_positions`, then pass its actual integer `positionId` to `close_position_link`. |
| Cancel or change a pending order | Identify the current pending order with `get_pending_orders`, then pass its actual integer `orderId` to `cancel_order_link`. |

Use records from the connected account and ask the user to choose when the target is ambiguous. Do not invent IDs, treat a historical order as pending, or choose an arbitrary position. The link tools check the current account lists. `close_position_link` rejects an unknown, closed or not-yet-filled position; `cancel_order_link` requires a current pending order. For a stale target, recheck its list and explain its current state; do not construct a replacement URL manually.

These tools take only the target ID. They open the position or pending-order modification screen; they do not prefill new SL/TP values, a partial-close size or other requested amendments. Tell the user to enter those choices in cTrader and confirm there. State that the call closed or cancelled nothing and changed no protection or order parameter.

If the requested end states conflict, such as fully closing a position and leaving it open with a new stop loss, clarify the intended result. You can supply the non-executing modification link while that choice is pending; do not describe either action as completed or prefilled.

## Account selection and returned notices

Return the server-provided URL unchanged, including its account selector when present. The server can add `acc` for the connected account. If it cannot obtain that account number, the result warns that the link will open on the account active in cTrader. Preserve the warning and tell the user to check the account before confirming. Do not promise account targeting when the result says it is unavailable.

Preserve the volume notice and any warning about an existing opposite-side position. The server's new-order warning is advisory; a missing warning does not prove that no opposite position exists. Opening a new order still does not close one.

If link generation is not configured, an ID or symbol is unavailable, or the server reports an error, explain that result without inventing a working link. Do not open links to submit actions, press Place, Close or Cancel, save amendments, or use direct trading tools. The user performs any trading action in cTrader.
