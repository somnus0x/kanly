---
name: breakdown
description: Create file-level task breakdowns for non-trivial changes. Use when planning R1 or R2 implementation, or when user says "breakdown", "task plan", "what files change", "implementation plan". Use proactively after a design spec is approved and before implementation begins. Do NOT use for R0 changes.
argument-hint: [write|view|validate]
---

# Breakdown — File-Level Task Plans

Break R1/R2 work into concrete, dispatchable tasks before touching code. Not 2-minute micro-steps — file-level plans that another agent could execute without asking questions.

---

## What This Skill Does

When a feature touches multiple files, the agent makes scope decisions on the fly — which file first, what changes where, what depends on what. Those decisions are invisible. If something breaks, you can't trace which decision was wrong. This skill makes the plan explicit before implementation starts.

## When to Use

| Reversibility | Breakdown? | Why |
|---|---|---|
| **R0** | No | Small enough to hold in your head |
| **R1** | **Yes** | Multiple files, dependencies between changes, worth planning |
| **R2** | **Yes** | Critical changes — every file touch should be deliberate |

---

## Commands

### `breakdown write`

Create a task breakdown for the current work.

1. Confirm a design spec exists (`/design check`). If not, gate.
2. Read the spec to understand scope
3. List every file that will be created, modified, or deleted
4. Order them by dependency (what must change first)
5. For each file, describe the specific change
6. Add a verification step for each task

**Output format:**

```markdown
# Breakdown: <Feature Name>
**Spec:** <link to design spec>
**Date:** <today>
**Reversibility:** R1/R2

## Tasks (in order)

### 1. <file path>
**Action:** create | modify | delete
**Change:** <what specifically changes in this file>
**Depends on:** <task number, or "none">
**Verify:** <how to confirm this task is correct>

### 2. <file path>
**Action:** modify
**Change:** <specific change>
**Depends on:** Task 1
**Verify:** <verification step>

...

## Summary
- Files touched: N
- New files: N
- Deleted files: N
- Estimated scope: <small|medium|large>
```

### `breakdown view`

Show the current task breakdown.

1. Read the breakdown doc for the current work
2. Show completion status per task (done/pending)
3. Highlight the next task to execute

### `breakdown validate`

Check the breakdown for completeness and quality.

1. Read the breakdown
2. Check for anti-patterns:
   - **Placeholders** — "TBD", "TODO", "add appropriate handling" → FAIL
   - **Vague changes** — "update the file" without specifics → FAIL
   - **Missing dependencies** — Task 3 needs Task 1's output but doesn't declare it → WARN
   - **No verification** — A task without a verify step → FAIL
3. Report pass/fail with specific issues

---

## The No-Placeholders Rule

Every task in the breakdown must be concrete enough that another agent — with no conversation context, no memory of previous sessions — could execute it correctly. If a task says "add appropriate error handling," that's a placeholder. Specify WHAT error handling.

**Good:** "Add try/catch around the API call in `fetchMarketData()`. On failure, log the error and return the cached data from `data/market/cache.json`."

**Bad:** "Add error handling to the fetch function."

---

## Rules

- **File-level, not line-level.** This isn't superpowers' 2-minute micro-tasks. One task per file or logical unit.
- **Order matters.** Dependencies between tasks must be explicit. If task 3 needs task 1's output, say so.
- **No placeholders.** If you can't describe the change concretely, the spec isn't detailed enough. Go back to `/design write`.
- **Verification per task.** Every task has a "how to confirm it's correct" step.
- **Breakdowns are dispatchable.** Another terminal should be able to pick up a task and execute it from the breakdown alone.
- **R0 is exempt.** Don't create breakdowns for trivial changes.

---

## Optional Integration

- Before `breakdown write` → `/design check` to verify spec exists
- After breakdown is approved → implement task by task
- After each task → `/verify run` to confirm the verification step
- After all tasks → `/test-gate` to check coverage
- After all tasks → `/review write` to create a review dispatch
- If breakdown reveals scope creep → `/scope check`

These are recommendations, not requirements. This skill works fully standalone.
