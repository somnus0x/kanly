---
name: review
description: Cross-terminal spec compliance and code quality review for R1/R2 changes. Use after an R1 or R2 change ships, or when user says "review", "cross-terminal review", "spec compliance", "check this change". Use proactively after R1 changes land to suggest writing a review dispatch. Creates audit trail in vault or handoff system.
argument-hint: [write|check|done]
---

# Review — Cross-Terminal Spec Compliance

After R1 changes ship, write a review dispatch so another terminal can verify spec compliance and code quality. The agent that built it shouldn't be the only one who checks it.

---

## What This Skill Does

Three terminals working on the same codebase means three agents making independent decisions. When one terminal ships an R1 change, the others have no structured way to verify it matches the spec and doesn't introduce quality issues. This skill creates that verification loop.

The reviewing terminal gets: the spec, the diff, and a structured check. It's not a rubber stamp — it's a fresh pair of eyes with isolated context.

## When to Use

| Reversibility | Review? | Why |
|---|---|---|
| **R0** | No | Too small to justify |
| **R1** | **Yes** | Costly to reverse — worth a second check |
| **R2** | **Optional** | Already requires human approval via `/guard`. Review adds value but R2 is already gated. |

---

## Commands

### `review write`

Create a review dispatch after an R1 change ships.

1. Identify the change that was just completed
2. Find the spec it was built against (`/design check` or from conversation)
3. Generate a review dispatch containing:

```markdown
# Review: <Change Name>
**Date:** <today>
**Reversibility:** R1
**Author terminal:** <which terminal made the change>

## Spec
<paste or link the design spec this was built against>

## Changes
<summary of what was changed — files, functions, endpoints>

## Diff
<key sections of the diff, or instructions to run `git diff <commit>` >

## Review checklist
- [ ] Does the code match the spec? (not more, not less)
- [ ] Are there untested edge cases?
- [ ] Does this increase coupling between modules?
- [ ] Are there hardcoded assumptions that should be configurable?
- [ ] Would this break if the spec assumptions change?
```

4. Save as a handoff dispatch:
   - Solo mode: `~/.claude/handoffs/review-<name>.md`
   - Team mode: `<repo>/.claude/handoffs/<author>--review-<name>.md`

### `review check`

Read pending review dispatches.

1. Check `~/.claude/handoffs/` (solo) or `<repo>/.claude/handoffs/` (team) for review dispatches
2. For each pending review:
   - Read the spec
   - Read the diff (or run the diff command)
   - Walk through the review checklist
   - Report findings:

```
REVIEW: <Change Name>

SPEC COMPLIANCE:
  ✓ <what matches>
  ✗ <what drifts from spec>
  ? <what can't be verified from code alone>

CODE QUALITY:
  ✓ <clean patterns>
  ⚠ <concerns — coupling, assumptions, missing tests>

VERDICT: <APPROVED | CONCERNS | BLOCKED>
<Summary of findings>
```

3. If concerns found, list specific file:line references

### `review done`

Archive a completed review.

1. Delete the review dispatch file (git history preserves it)
2. Optionally save a summary to the vault audit trail:

```markdown
## Review: <name> — <date>
**Verdict:** APPROVED / CONCERNS (N resolved)
**Key findings:** <1-2 bullet summary>
```

---

## Review Quality

A good review catches:

- **Spec drift** — code does more or less than what was specified
- **Silent coupling** — new imports or dependencies between previously independent modules
- **Untested paths** — changed behavior without corresponding test updates
- **Hardened assumptions** — magic numbers, hardcoded values, environment-specific logic
- **Missing error handling** — new failure modes introduced without recovery paths

A bad review is:

- "Looks good" without checking the spec
- Nit-picking style when the question is correctness
- Approving without reading the diff
- Rubber-stamping because it's the same agent that wrote the code

---

## Rules

- **Different context reviews.** The reviewing terminal should have fresh context, not the build context. That's why this is a dispatch, not a self-review.
- **Spec is the standard.** The review checks against the spec, not against the reviewer's preferences. If the code matches the spec and the spec is wrong, fix the spec.
- **R1 only by default.** Don't create review overhead for R0 changes. R2 already has human-in-the-loop via `/guard`.
- **Review is a conversation, not a gate.** Concerns are surfaced, not blocking. The author terminal decides how to address them.
- **Audit trail matters.** Save review summaries. They're useful for retros and pattern detection.

---

## Optional Integration

- Before `review write` → the change should have passed `/test-gate`
- During `review check` → run `/dissent review` for additional adversarial analysis
- If review finds spec drift → `/spec diff` to see if spec was updated
- If review finds coupling → `/dissent` to formalize the concern
- After review → `/verify run` on the full test suite
- Archive review summaries → `/learn add` if review revealed a pattern

These are recommendations, not requirements. This skill works fully standalone.
