# Trade classification contract

TradingLab separates research classification into three dimensions with different meanings and cardinalities. Definitions are managed vocabularies, not free-text labels.

| Dimension | Question | Cardinality per trade | Analytics role |
| --- | --- | --- | --- |
| Playbook | Why did I click? | `0..1` | Mutually exclusive primary performance bucket |
| Confluence | What else was present at the level? | `0..N` | Secondary filter or conditional-analysis dimension |
| Tag | What was going on with me or the execution? | `0..N` | Qualitative metadata only |

## Playbook

A playbook is the primary setup or reference that caused the trade level to be identified before price arrived. A trade can have no playbook while it is awaiting classification, but the database must never allow more than one.

Multiple playbooks are forbidden because playbook statistics must be mutually exclusive. When several references coincide, such as VWAP, POC, LVN, or a VWAP band, the reason that actually identified the level is the playbook and the other references are confluences. If the primary reason is unclear, keep the trade unclassified and record the ambiguity with a tag. A recurring combination that proves to be a distinct setup should become its own single playbook.

## Confluences and tags

Confluences are atomic market conditions that accompanied the primary setup. Store coincident conditions as separate confluences rather than combined labels such as `POC + LVN`.

Tags describe mistakes, psychology, execution circumstances, ambiguity, or other qualitative review conditions. Tags do not replace a playbook and must not be used as primary performance buckets. The existing optional tag category may organize this qualitative vocabulary, but it does not affect classification state or analytics semantics.

## Classification state

A trade is **Classified** only when it has one playbook. It is **Unclassified** when it has no playbook. Confluences, tags, and Notes do not change that state. Removing a playbook immediately returns the trade to the unclassified review population.

## Retirement

Playbook, confluence, and tag definitions have an active state. Inactive definitions are retired: they are hidden from new assignments by default, remain visible on historical trades where already attached, can be removed from those trades, and can be reactivated. Normal application workflows retire definitions rather than deleting them so historical meaning is preserved.

## Responsibility boundaries

The database owns structural cardinality, referential integrity, replacement semantics, and atomic persistence. In particular, it guarantees at most one playbook relationship per trade and saves the playbook, confluences, tags, and Notes together.

The frontend represents the cardinalities directly, offers only active definitions for new assignments, preserves attached inactive definitions until removed, and determines Classified/Unclassified state only from the playbook relationship. Relationship loading remains batched.

The importers and source-data producers do not create or infer these research classifications.

## Analytics semantics

Future statistics may group trades by the single playbook, then filter or condition those results by confluences. Tags remain qualitative metadata and must not be counted as mutually exclusive setup categories.
