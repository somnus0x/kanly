# Changelog

## [0.2.0] — 2026-03-19

### Added

- **`/guard trace <symbol>`** — Map blast radius before changing anything. Finds all callers, type consumers, test references, and cross-repo exposure. Auto-classifies risk based on radius size.
- **`/guard verify <symbol>`** — Verify removed/replaced code is dead. Reports DEAD/ALIVE/GHOST with survivor locations. Required step after any replacement or deletion.
- **`/handoff verify`** — Diff handoff API contract tables against actual codebase. Reports MATCH/DRIFT/MISSING/UNDOCUMENTED per endpoint.
- **`/replace`** — New standalone skill. Kill-and-prove protocol for full replacements. Orchestrates confirm → guard trace → execute → guard verify → report. Includes migration map template for data source swaps.

### Changed

- Guard description updated with new trigger words: "blast radius", "trace", "what calls this", "is this dead", "verify removal"
- Handoff description updated with new trigger words: "verify contract", "api drift", "does this match the handoff"
- README updated: six → seven protocols, new usage examples, updated flow diagram
- CLAUDE_SNIPPET updated with new skill references and proactive trigger examples

## [0.1.0] — 2026-03-08

### Added

- Initial release: handoff, spec, scope, learn, guard, dissent
