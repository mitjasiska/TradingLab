# ATAS fill lifecycle

ATAS exposes fills through more than one path:

- historical rows can be enumerated from `TradingManager.MyTrades`;
- filtered rows can be enumerated from `FilteredStatistics.MyTrades`;
- fills can arrive through `NewMyTrade` and filtered-statistics callbacks.

The important observed behavior is that ATAS can enumerate a historical row through `TradingManager.MyTrades` and later emit that same row through `NewMyTrade`, even when `FilteredStatistics.MyTrades` is empty. A direct callback is therefore not proof that a fill is new or live.

TradeStatistics and TradeVideoCapture are independently hardened against this lifecycle. At startup they seed the relevant historical identities before admitting live fills, and they deduplicate fills delivered through multiple callback paths.

This is a cross-project constraint because a mismatch can create a structured trade without the corresponding video, reopen an old trade, or break TradeId alignment.

Automated lifecycle tests are necessary but not sufficient. Acceptance for changes in this area includes a real ATAS smoke test covering startup history, a new live or Replay trade, and CSV/video TradeId matching.

Revalidate the enumeration and callback behavior during any future ATAS or ATAS X migration.
