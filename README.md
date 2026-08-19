# TradingLab

TradingLab is the system that turns executed ATAS trades into structured research data, synchronized trade videos, and a web-based review workflow.

This repository is the human-facing overview and coordination point for that system. It contains no application code. Each implementation repository remains responsible for its own technical and product documentation.

## How the system fits together

```text
ATAS
  |-- TradeStatistics CSV ------------> statistics importer --|
  `-- TradeVideoCapture MP4/JSON ------> video importer --------|--> Supabase --> TradingLab Vue application
```

TradeStatistics and TradeVideoCapture observe the same trading activity independently. The statistics import creates sessions and trades first; the video import then finds those trades by TradeId and links their recordings.

## Repository responsibilities

| Repository | Responsibility |
| --- | --- |
| `C:\development\projects\AtasIndicators` | Owns `TradeStatistics`, the structured trade-data producer. |
| `C:\development\projects\TradeVideoCapture` | Owns trade-video capture and MP4/JSON production. |
| `C:\development\projects\TradingResearchData` | Owns database migrations, both importers, and import orchestration. |
| `C:\development\projects\trading-research-app` | Owns the Vue application and trade-review interface. |

Build instructions, implementation details, product features, and project-specific deployment instructions belong in those repositories.

## Operator workflow

```text
trade -> capture -> one-click import -> TradingLab -> review/classify
```

The statistics importer owns session and trade ingestion. The video importer uploads and links video only after finding an imported trade with the same TradeId. The one-click wrapper sequences those operations but contains no business logic.

Classification and reference-data decisions happen in TradingLab, not through interactive import prompts.

## Shared runtime locations

```text
C:\TradeExport\statistics
C:\TradeExport\video
C:\TradeExport\video\trades\yyyy-MM-dd\<trade_id>.mp4
C:\TradeExport\video\trades\yyyy-MM-dd\<trade_id>.json
```

The first two paths are the shared default roots. Rolling-capture internals remain owned by TradeVideoCapture.

## Cross-project guarantees

### One trade identity

The same deterministic identity connects all representations of a trade:

```text
TradeStatistics CSV TradeId
= TradeVideoCapture JSON trade_id
= MP4 basename and identity
```

The exact algorithm and change constraints are documented in [TradeId](docs/trade-id.md).

### Historical ATAS fills are not necessarily live fills

ATAS can enumerate historical fills and later deliver them through a direct callback even when filtered statistics are empty. Both producers are hardened against treating those rows as new trades. This behavior must be rechecked during a future ATAS or ATAS X migration.

The lifecycle and acceptance implications are documented in [ATAS fill lifecycle](docs/atas-fill-lifecycle.md).

### Session names are editable and explicitly generated

Users can edit a session name or explicitly regenerate it from classified session parameters. Re-imports reuse sessions already linked by deterministic TradeIds. The behavior and generated-name format are documented in [Session names](docs/session-names.md).

### Trade research has three distinct classification dimensions

Playbooks identify the single primary reason for a trade, confluences capture additional market references, and tags capture qualitative review conditions. Classified/Unclassified state depends only on the playbook. The cardinalities and responsibility boundaries are documented in [Trade classification](docs/trade-classification.md).

## Current high-level status

As of 2026-08-17:

- **TradeStatistics:** working; historical-fill lifecycle hardened; real CSV/video TradeId matching verified.
- **TradeVideoCapture:** working; historical-fill lifecycle hardened; rolling capture and export working; real TradeId matching verified.
- **Import layer:** statistics and video importers exist; tested one-click statistics-then-video orchestration exists.
- **TradingLab frontend:** exists and is deployed; details belong in its own README.

This section is a snapshot, not a changelog.

## Documentation model

```text
Human-facing system overview -> this README
Detailed cross-project concept -> docs
Repository-specific fact -> that repository's README
```

Do not maintain two authoritative copies. Prefer references over copied sections.

Deeper system documentation:

- [Deterministic TradeId contract](docs/trade-id.md)
- [ATAS fill lifecycle](docs/atas-fill-lifecycle.md)
- [Session name contract](docs/session-names.md)
- [Trade classification contract](docs/trade-classification.md)
- [Cross-project architecture decisions](docs/decisions.md)

## Development loop

```text
observe real behavior -> identify one narrow task -> inspect before editing -> implement
-> automated tests -> Release build -> deploy if applicable -> real acceptance test
-> report result -> commit -> freeze component
```

Automated tests are not sufficient evidence for ATAS lifecycle behavior. Real ATAS smoke testing is part of acceptance.
