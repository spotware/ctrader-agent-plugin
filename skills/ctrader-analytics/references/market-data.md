# Market data

## Symbols and quotes

Resolve a requested symbol through `get_symbols` and reuse its numeric `symbolId` for the same account. Match names exactly, including case and broker suffixes; do not silently substitute another instrument. If a connected tool's current schema explicitly accepts `symbolName`, it may be used directly. Never send `symbolName` to an ID-only tool.

Batch the requested symbol IDs into the `symbolId` array for `get_spot_prices` and match returned rows by `symbolId`. Supply at least one symbol across the identifier fields the connected schema accepts. A quote may omit one side. Without both bid and ask, do not calculate the spread or fill in a missing side from another instrument.

Check `error` on each returned `prices` entry before reading its prices. The server can return successful quotes alongside entries with an error and null price fields. Report the successful rows and identify unavailable symbols; a successful tool call does not establish that every requested quote succeeded. Keep null or missing values unavailable rather than treating them as zero.

| Per-symbol error | Recovery |
| --- | --- |
| `SYMBOL_NOT_FOUND` | The ID may be unknown or its subscription may not have been confirmed in time. Check the ID, or use the exact name if the schema supports `symbolName`, then retry once. |
| `PRICE_TIMEOUT` | No quote arrived in time, for example during a market closure. Report it as unavailable and retry later if needed. |
| `SUBSCRIPTION_CONFLICT` | A parallel request for the same symbol was running. Retry without overlapping calls for that symbol. |

An unknown `symbolName` can fail the whole call. Correct the name from the account's symbol catalogue without substituting a different instrument. Keep retries bounded and report unresolved errors.

Divide bid, ask, session high, session low and session close by **100000**. For example, raw USDJPY `15731000` displays as `157.31`; raw XAUUSD `399327000` displays as `3993.27`. `digits` controls display precision, not the wire scale. There is no `pipDigits` field to use as the denominator.

The absolute spread is `(ask - bid) / 100000`. Express it in pips only when the symbol's `pipPosition` is available: pip size is `10^(-pipPosition)`. Quote timestamps are epoch milliseconds. Report stale, missing or inconsistent quotes instead of presenting them as current market conditions.

## Candles

The MCP `get_trendbars` schema currently exposes `M_1`, `M_5`, `M_15`, `M_30`, `H_1`, `H_4`, `D_1`, `W_1` and `MN_1`. Follow the connected schema if it changes. Do not pass an unsupported period because a different API lists it. Offer an available interval or explicitly labelled local aggregation when appropriate.

Pass the resolved `symbolId` to `get_trendbars`. If its connected schema also accepts `symbolName`, pass exactly one identifier form. Use timestamp strings containing a valid ISO 8601 date-time with `Z` or an explicit offset, or integer epoch milliseconds as a string. Prefer UTC ISO 8601 strings. Date-only values, date-times without a zone, seconds and microseconds are invalid. The supported bounds are `1000000000000` through `4102444800000` milliseconds, inclusive. Do not send a JSON number to a parameter declared as a string or guess a different date when validation fails.

With only `count`, the server searches back at most 30 days from its current time. `count` limits results; it does not extend that interval. The default is 100 candles and the server clamps the requested row count to 1000.

For an explicit range, supply both `fromTimestamp` and `toTimestamp`, with the start before the end and a duration no greater than 30 days per request. Split longer ranges into adjacent windows. Decode OHLC by dividing by **100000**. Leave candle `volume` as a tick count, not shares, lots or traded cash.

Deduplicate candles by symbol, period and timestamp, then sort chronologically. A full row limit can hide older candles because the server counts backward from the end of the window; narrow or subdivide the window instead of assuming it is complete. Do not promise N candles when market closures or the available history yield fewer.

The response can include bars overlapping the requested boundaries. An H_1 query from 14:22 yesterday to 14:22 today can include 25 bars, starting at 14:00 yesterday and ending with today's unfinished 14:00 bar. State the returned opening-time coverage and identify incomplete boundary bars. Highs and lows over those whole bars are not exact highs and lows for the requested minute-to-minute interval; do not label them that way. Use finer data if exact boundaries matter and it is available.

Bound concurrent reads, honour throttling and stop when coverage cannot be verified. Do not embed a fixed requests-per-second guarantee in the analysis.
