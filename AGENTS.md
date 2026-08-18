# AI agent guidance

## TradingLab system authority

- This documentation-only repository is the architectural and coordination repository for the larger TradingLab system. `mitjasiska/TradingLab` is the source of truth for cross-repository architecture and integration contracts.
- Before changing system documentation or implementation in a way that affects another repository, inspect the relevant TradingLab documentation and the affected implementation repositories. Cross-repository boundaries include database and Supabase schemas, CSV formats, trade or session identifiers, instrument naming, video naming or storage, importer and API contracts, shared terminology, deployment assumptions, and component responsibilities. Do not endorse a locally convenient change that silently breaks another component.
- Use this authority order: explicit user instructions; TradingLab system documentation for cross-repository architecture and contracts; repository `AGENTS.md`; repository-specific documentation; existing implementation. If documentation and implementation disagree, identify the inconsistency instead of silently choosing one.
- Each implementation repository's `AGENTS.md`, README, source code, and local documentation remain authoritative for local-only implementation details. TradingLab documentation does not override valid repository-specific instructions unless an actual cross-repository conflict exists, and local-only refactoring does not require unnecessary system analysis.

1. Read `README.md` first. Read the relevant file under `docs` when the task touches a deeper cross-project concept.
2. Keep this repository documentation-only. It owns system-wide TradingLab architecture, coordination, contracts, shared paths, and cross-project status.
3. Inspect the relevant implementation repository before describing current behavior:
   - `C:\development\projects\AtasIndicators`
   - `C:\development\projects\TradeVideoCapture`
   - `C:\development\projects\TradingResearchData`
   - `C:\development\projects\trading-research-app`
4. Never invent current architecture or infer it from stale documentation when source or newer evidence is available.
5. Keep one authoritative source per level:
   - human-facing system overview -> `README.md`;
   - detailed cross-project concept -> the relevant file under `docs`;
   - repository-specific fact -> that repository's README.
6. Prefer references over copied sections. Do not copy system architecture into every project README or project details into this repository.
7. Update `README.md` when repository responsibilities, shared runtime paths, data flow, system status, or the human-facing system overview changes. Update the relevant `docs` file when a deeper cross-project contract or lifecycle changes.
8. Do not update it for internal refactors, local bug fixes, UI details, individual tests, or project-specific deployment details.
9. Add a decision record directly under `docs` only for important cross-project decisions with lasting consequences.
10. Prefer small documentation changes tied to verified implementation changes.

Do not modify an implementation repository unless the task explicitly includes it. Inspect the worktree and final diff before handing off any change.
