---
name: test-gate
description: Enforce test coverage for non-trivial changes before completion. Use when R1 or R2 changes are about to be marked complete, or when user says "test gate", "are there tests", "test coverage", "test check". Use proactively before commits on R1/R2 changes to verify test coverage exists. Do NOT enforce for R0 changes.
argument-hint: [check|run|gate]
---

# Test Gate — Tests Required Before Done

R1 changes require at least one test covering the changed behavior before marking complete. Not TDD. Not full coverage. Just proof that the change is testable and tested.

---

## What This Skill Does

Tests happen when they happen — which means they often don't happen. This skill adds a gate: before an R1 change is marked complete, at least one test must cover the changed behavior. Not test-first (TDD), not 100% coverage — just "does a test exist that would catch a regression?"

## When to Gate

| Reversibility | Test required? | What's checked |
|---|---|---|
| **R0** | No | Move fast |
| **R1** | **Yes** | At least one test covering the changed behavior |
| **R2** | **Checklist** | Add "are there tests?" to the human review checklist. Not auto-enforced — R2 already requires explicit approval. |

---

## Commands

### `test check`

Find tests related to the current changes.

1. Identify changed files (from `git diff` or conversation context)
2. For each changed file, search for corresponding test files:
   - `*.test.ts`, `*.spec.ts`, `*_test.py`, `test_*.py` etc.
   - Same directory or `__tests__/` / `tests/` directory
3. For each test file found, check if any test exercises the changed function/endpoint/behavior
4. Report:

```
TEST CHECK: <feature/change>

COVERED:
  ✓ <changed-file:function> — tested by <test-file:test-name>

UNCOVERED:
  ✗ <changed-file:function> — no test found

COVERAGE: N/M changed behaviors have tests
```

### `test run`

Run the relevant tests and show output.

1. Identify the test command for this project (npm test, pytest, etc.)
2. Run tests relevant to the changed files (not the full suite unless needed)
3. Show full output via `/verify run`
4. Report pass/fail

### `test gate`

Final gate check before marking an R1 change as complete.

1. Run `test check` to find coverage
2. If any changed behavior is uncovered:
   - **BLOCKED** — "R1 change has untested behavior. Write a test or acknowledge the gap."
   - List specifically what needs a test
3. If all changed behaviors have tests:
   - Run `test run` to verify they pass
   - If passing → **CLEAR** — "Test gate passed. Change can be marked complete."
   - If failing → **BLOCKED** — "Tests exist but are failing."

---

## What Counts as "Covering the Changed Behavior"

A test covers the changed behavior if:

- It calls the changed function/endpoint/component
- It asserts on the behavior that was modified (not just that the function exists)
- It would fail if the change were reverted

A test does NOT cover the changed behavior if:

- It tests a different function in the same file
- It only tests the happy path when the change affects error handling
- It's a snapshot test that auto-updates (proves nothing)

---

## Rules

- **One test minimum for R1.** Not zero, not full coverage. One real test that exercises the changed behavior.
- **The test must be meaningful.** A test that always passes regardless of the implementation is not a test.
- **R0 is exempt.** Don't slow down trivial changes with test requirements.
- **R2 is a checklist item, not a gate.** R2 changes already go through human review via `/guard`. Add "are there tests?" to that conversation, but don't auto-block.
- **Override is fine.** If the operator says "ship without tests," respect it. Log the decision via `/learn add`.
- **Show the output.** Use `/verify run` to prove the tests pass. "Tests pass" without output is not verified.

---

## Optional Integration

- Before `test gate` → `/guard classify` to confirm R1/R2
- After tests written → `/verify run` to prove they pass
- If test reveals a gotcha → `/learn add`
- After test gate passes → `/review write` to create a review dispatch
- If untested and overridden → `/learn add` to log the decision

These are recommendations, not requirements. This skill works fully standalone.
