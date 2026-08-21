# Trade classification contract

TradingLab separates research classification into four independent dimensions with different meanings and cardinalities. Definitions are managed vocabularies, not free-text labels.

| Dimension | Question | Cardinality per trade | Analytics role |
| --- | --- | --- | --- |
| Setup | What trading pattern or opportunity did I take? | Exactly `1` after review; absent only while Unclassified | Mutually exclusive primary performance bucket |
| Market Regime | What market regime did I believe was present for this trade? | `0..1` | Independent structured dimension for future statistical filtering and grouping |
| Confluence | What else was present at the level? | `0..N` | Secondary filter or conditional-analysis dimension |
| Tag | What was going on with me or the execution? | `0..N` | Mistakes, psychology, circumstances; qualitative metadata only |

## Setup

A Setup is the trading pattern or opportunity associated with a trade, such as `VWAP Continuation`, `Band Continuation`, `Band Fade`, `PDL Rejection`, or `PDH Rejection`. Setup definitions are independent research entities: trades reference them, and future workflows may record opportunities that were not traded without introducing another trading-pattern concept.

Every completed trade review requires exactly one Setup. An imported trade can temporarily have no Setup only while awaiting classification; the database must never allow more than one assignment, and the research save operation rejects a missing Setup. When several market conditions coincide, the trading pattern is the Setup and the other conditions are Confluences.

## Market Regime

Market Regime records the regime the trader believed was present for that individual trade. It is per trade because the assessment can change during a session; it must not be inferred later from how the session developed or from trade results.

A trade has zero or one Market Regime. The fixed values are exactly:

- `Ranging`
- `Mean Reverting`
- `Undecided`

`NULL` means the trade has not yet been classified for Market Regime. `Undecided` means the trader explicitly assessed the regime and concluded it was undecided. These states are distinct: existing trades remain `NULL` until manually reviewed, and the application must not default or backfill them to `Undecided`.

Market Regime is independent of Setup, Confluences, and Tags. It is structured research metadata, not a tag, and may later support statistical filtering, grouping, expectancy comparisons, and MAE/MFE comparisons.

## Confluences and tags

Confluences are atomic market conditions that accompanied the primary setup. Store coincident conditions as separate confluences rather than combined labels such as `POC + LVN`.

Tags describe mistakes, psychology, execution circumstances, ambiguity, or other qualitative review conditions. Tags do not replace a Setup and must not be used as primary performance buckets. The existing optional tag category may organize this qualitative vocabulary, but it does not affect classification state or analytics semantics.

## Classification state

A trade is **Classified** only when it has one Setup. It is **Unclassified** while its Setup is missing. Market Regime, Confluences, Tags, and Notes do not change that state. Once assigned, Setup is required: it can be replaced with another Setup but cannot be cleared by the research save workflow.

## Retirement

Setup, Market Regime, Confluence, and Tag definitions have an active state. Inactive definitions are retired: they are hidden from new assignments by default, remain visible on historical trades where already attached, can be replaced or removed where their dimension permits, and can be reactivated. Normal application workflows retire definitions rather than deleting them so historical meaning is preserved. Setups, Confluences, and Tags can be created and edited through Research Settings. Market Regime names are fixed; settings may only retire or reactivate the three seeded values.

## Responsibility boundaries

The database owns structural cardinality, referential integrity, replacement semantics, and atomic persistence. In particular, it guarantees at most one Setup relationship, requires a Setup when research is saved, permits one nullable Market Regime reference, and saves Setup, Market Regime, Confluences, Tags, and Notes together.

The frontend represents the cardinalities directly, offers only active definitions for new assignments, preserves an attached inactive Setup until it is replaced, and determines Classified/Unclassified state only from the Setup relationship. Setup uses a required single-select. Market Regime uses a clearable single-select with no free-text creation or default. Relationship loading remains batched.

The importers and source-data producers do not create or infer these research classifications.

## Analytics semantics

Statistics group trades by their single Setup, then may filter or condition those results by Market Regime and Confluences. Tags remain qualitative metadata and must not be counted as mutually exclusive setup categories.
