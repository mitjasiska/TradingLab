# Adopt Setup as the trading-pattern domain term

## Context

TradingLab used Playbook for the single trading pattern or opportunity associated with a trade. That term overlapped with common meanings for a collection of strategies and made future support for untraded opportunities ambiguous.

## Decision

Setup is the only trading-pattern domain concept. Setup definitions are independent entities, trades reference at most one while awaiting review and exactly one after research is saved, and no parallel alias is exposed by the database API or frontend.

Migration 014 renames the existing PostgreSQL tables, columns, constraints, indexes, policies, relationships, and RPC parameter in place. Historical assignments retain their identifiers and timestamps. The former identifiers remain only in already-applied SQL history and at migration 014's one-way source boundary.

## Consequences

- `TradingResearchData` owns `setups`, `trade_setups`, `setup_id`, and the `p_setup_id` research RPC parameter.
- `trading-research-app` uses Setup in models, queries, settings, review, status, and tests.
- Market Regime, Confluence, and Tag semantics do not change.
- ATAS statistics and video producers are unaffected because they do not create or consume research classifications.

## Affected repositories

- `TradingLab`
- `TradingResearchData`
- `trading-research-app`
