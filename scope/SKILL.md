---
name: scope
description: Manage project non-goals and scope boundaries. Use when user says "scope", "non-goal", "out of scope", or when current work might be drifting beyond what the project intends to build. Helps teams stay focused by declaring what they are NOT building. Use proactively during planning to check proposed work against declared non-goals before finalizing the plan (run `scope check`).
argument-hint: [add <non-goal>|check|list]
---

# Scope — Exclusion Zones

Manage what this project explicitly does NOT do. Every project has finite resources — declaring non-goals protects focus.

**File:** `<project-root>/spec/NON_GOALS.md`

---

## What This Skill Does

Projects fail from scope creep more than from missing features. This skill maintains a list of declared non-goals — things the team has explicitly decided NOT to build — and checks if current work is drifting into those areas.

## Commands

### `scope add <non-goal>`

Declare something as out of scope.

1. Read current `spec/NON_GOALS.md` (create if it doesn't exist)
2. Add the non-goal with a one-line rationale
3. Use the format below
4. No approval gate — non-goals are declarations, not code changes

**Format:**

```markdown
## <Non-goal title>
<What we're NOT doing and why>
<!-- added: <date> -->
```

### `scope check`

Check if current work drifts into non-goal territory.

1. Read `spec/NON_GOALS.md` and extract all declared non-goals
2. Look at the current session context — what's being worked on
3. If inside a git repo, also check recent changes: `git diff --name-only HEAD~5`
4. Compare current work against declared non-goals
5. Report with confidence signals:
   - **No drift detected** — no semantic overlap found (does not guarantee absence of drift)
   - **Possible drift** — work is adjacent to a non-goal, may or may not constitute drift. Surface for user judgment. Do NOT block.
   - **Drift — high confidence** — current work directly implements a declared non-goal. Stop and confirm with user.

### `scope list`

List all current non-goals, concisely.

1. Read `spec/NON_GOALS.md`
2. Print each non-goal as a one-liner: title + rationale
3. Show count and last modified date

---

## Rules

- Non-goals are **guardrails, not permanent walls** — they can be revisited
- Adding a non-goal is free — removing one should be deliberate (discuss with user)
- Do not block work on "possible drift" — surface it and let the user decide
- Non-goals protect focus — they're not about what's impossible, but what's not worth doing *now*
