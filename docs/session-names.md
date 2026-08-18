# Session name contract

`sessions.name` is the single session label displayed and edited by TradingLab. There is no separate display-title field.

## Import-created names

The statistics importer keeps its two existing naming paths:

- A single-file import with complete metadata uses its configured name or generates a name from market date, instrument, raw session window, mode, and optional replay number.
- Pending transport imports, which cannot know the final session window, mode, or run, use an `Imported` fallback containing the market date, account, and instrument.

These initial names are valid even when they are not yet the final research name. The frontend can later classify the session metadata and explicitly generate the normalized name.

Renaming does not break an ordinary re-import of the same CSV. Before looking up a session by name, the importer reuses the one session already linked to the CSV's deterministic TradeIds. It still rejects conflicting metadata, ambiguous duplicate-name matches, or TradeIds linked across multiple sessions.

## Editing behavior

The Session Detail **Edit Session** dialog edits `sessions.name` directly. A nonblank manual name is saved together with Session Window, Mode, and Run. Changing metadata never silently regenerates the name.

The dialog also offers **Generate session name**. This explicitly replaces the name field with a value derived from the current session parameters. The user must still select **Save**. Generation and save remain valid when the generated value is identical to the stored name.

Migration `TradingResearchData/database/010_allow_authenticated_session_name_updates.sql` grants authenticated browser users column-level update access to `sessions.name`; the existing row-level update policy continues to apply.

## Frontend-generated format

```text
YYYY-MM-DD INSTRUMENT SESSION_WINDOW MODE [#RUN]
```

Example:

```text
2026-08-13 NQ NY AM Replay #01
```

Generation uses these parameters:

| Segment | Source and formatting |
| --- | --- |
| Date | `market_date` in ISO `YYYY-MM-DD` form |
| Instrument | `instruments.code`, such as `NQ`, reached through `sessions.instrument_id` |
| Session window | Selected reference value, falling back to imported text; hyphens and underscores become spaces |
| Mode | Selected reference value, falling back to imported text |
| Run | Optional positive integer; prefixed with `#` and padded to at least two digits |

Whitespace is trimmed and collapsed. Date, instrument code, session window, and mode are required. The session relationship is authoritative: generation does not derive the instrument segment from `contracts.name`, `contracts.security_id`, trade text, or contract-name parsing. Run is optional, but when present it must be a positive whole number. If generation is unavailable, the user can still enter a valid name manually.

Historical inference and backfill of `sessions.instrument_id` are intentionally unsupported while existing data is disposable test data. Imports set the relationship directly for new sessions; an existing session without it is incomplete and must be recreated or corrected explicitly.

## Responsibility boundaries

- `TradingResearchData` owns the session schema, update permission, importer-created names, and re-import safeguards.
- `trading-research-app` owns manual name editing, explicit generation, validation, and display.
- `TradingLab/docs/session-names.md` is the cross-project authority for this contract.
