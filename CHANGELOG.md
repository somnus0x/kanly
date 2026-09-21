# Changelog

## [Unreleased]

### Changed

- **BREAKING (vocabulary): R0 and R2 swapped to match the operator's CLAUDE.md RULE 9.**
  Behaviour is unchanged; only the labels move.
  - `R0` was *fully reversible* and is now **irreversible / hard to reverse — STOP and get approval**.
  - `R1` is unchanged — *costly to reverse*.
  - `R2` was *hard or impossible to reverse* and is now **easily reversed — move fast, no gate**.

  Why: kanly and CLAUDE.md defined the same three tokens as exact opposites while loading into
  the same Claude Code session. Each was internally consistent, so neither looked wrong alone.
  The hazard was the label crossing a boundary — which is kanly's entire job. A `/handoff`
  dispatch reading "this is R2" meant *stop, hard to reverse* to kanly and *easily reversed,
  just do it* to CLAUDE.md. No incident is on record; this was fixed as a latent hazard.

  **If you installed skills before this release, re-copy them.** A mixed install is worse than
  either convention on its own.

## [0.3.1] — 2026-06-19

### Changed

- **Trimmed over-eager proactive triggers** on `guard`, `scope`, `spec`, `learn` so they stop auto-firing on routine work regardless of stakes. Protocol bodies unchanged — only *when they self-invoke*:
  - `guard` — fires proactively ONLY on high-stakes/R2 domains (money, schema, contract storage/ABI, public API, auth); never on routine R0/R1 work.
  - `scope` — on-demand only; no proactive plan-checking.
  - `spec` — active only where `spec/SPEC.md` exists; never prompts otherwise. `spec check` also flags BINDING rules the code has drifted from.
  - `learn` — `learn check` narrowed to "before debugging something that should-work-but-doesn't, or before a migration"; `learn add` still fires after non-obvious fixes.
- `dissent` not trimmed here — superseded by the surgical-dissent rework in 0.3.0. `handoff` (team-mode retained) and `replace` untouched.

## [0.3.0] — 2026-03-28

### Added

- **`/design-gate`** — Hard gate on R1/R2 implementation without a design spec. No spec, no code. R0 exempt. Forces architecture decisions to be conscious before code starts.
- **`/breakdown`** — File-level task breakdowns for R1/R2 changes. Concrete enough for another agent to execute without questions. No placeholders rule — every task specifies exact files, changes, and verification steps.
- **`/review`** — Cross-terminal spec compliance and code quality review. After R1 changes ship, creates a review dispatch for another terminal to verify. Audit trail in vault/handoff system.
- **`/test-gate`** — Test coverage requirement for R1 changes before marking complete. Not TDD — just "does a test exist that would catch a regression?" R2 adds to human review checklist. R0 exempt.
- **`/verify`** — Evidence-based completion claims. No "should work now" — run the command, show the output, then claim success. Referenced by test-gate, review, and all completion workflows.

### Changed

- **`/dissent` upgraded to surgical dissent** — Now traces decision trees branch by branch and self-answers from codebase before surfacing concerns to the operator. Reduces dissent fatigue by only asking questions code can't resolve. The five original conditions (coupling, hardening, blast radius, scope creep, convention violation) remain as the detection layer.
- README updated: seven → twelve protocols, new v2 pipeline diagram, updated flow diagram, new usage examples
- CLAUDE_SNIPPET updated with new skill references and proactive triggers for all five new skills

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
