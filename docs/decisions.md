# Cross-project architectural decisions

Use permanent decision records only for important TradingLab decisions that affect multiple repositories.

Examples include:

- changing the TradeId algorithm;
- changing storage provider or replacing Supabase;
- moving responsibility between importers;
- making a major change to the ATAS integration strategy.

Ordinary implementation choices, local refactors, bug fixes, and project-specific product decisions do not need an architecture decision record.

When a record is justified, add it directly under `docs` using the next sequential name:

```text
0001-short-title.md
```

Keep each record brief: context, decision, consequences, and affected repositories. Do not create speculative records.
