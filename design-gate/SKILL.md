---
name: design-gate
description: Enforce design-before-code for non-trivial changes. Use before any R1 or R2 change, or when user says "design", "spec first", "what's the plan", or when implementation starts without a written spec. Use proactively when you detect code being written for an R1/R2 change that has no corresponding spec document. Do NOT gate R0 changes — they move fast by definition.
argument-hint: [write|check|review]
---

# Design Gate — No Spec, No Code

Hard gate on implementation without design. If the spec doesn't exist, the code doesn't start.

---

## What This Skill Does

The most expensive bugs are architecture bugs — wrong abstractions, wrong boundaries, wrong assumptions baked into code before anyone questioned them. This skill enforces a simple rule: before writing code for anything costly to reverse, write down what you're building and why.

Not a 20-page design doc. Three sentences is fine for small changes. The point is forcing the decision to be conscious, not automatic.

## When to Gate

| Reversibility | Gate? | Why |
|---|---|---|
| **R0** (fully reversible) | No | CSS tweaks, logging, renames — just do it |
| **R1** (costly to reverse) | **Yes** | Config changes, dependency upgrades, new endpoints — write 3 sentences minimum |
| **R2** (hard to reverse) | **Yes** | Schema, contracts, money flows — full design doc with failure modes |

**The anti-pattern this kills:** "This is too simple to need a design." That's where unexamined assumptions cause the most rework.

---

## Commands

### `design write`

Create or append to a spec for the current work.

1. Determine the scope of the current change
2. Run `/guard classify` to confirm R1 or R2
3. If R0, say "R0 — no gate needed. Proceed." and stop.
4. Write a spec appropriate to the reversibility:

**R1 spec (minimum 3 sentences):**
```markdown
## <Feature/Change Name>
**Reversibility:** R1
**Date:** <today>

**What:** <what's changing>
**Why:** <why this approach over alternatives>
**Touches:** <files/systems affected>
```

**R2 spec (full design):**
```markdown
## <Feature/Change Name>
**Reversibility:** R2
**Date:** <today>

**What:** <what's changing>
**Why:** <why this approach over alternatives>
**Touches:** <files/systems affected>

### Failure modes
- <what goes wrong if this is wrong>
- <what's the rollback path>

### Alternatives considered
- <option B and why not>
```

5. Save to `spec/designs/<date>-<name>.md` or project-specific location
6. **Present to user for approval before proceeding**

### `design check`

Verify a spec exists for the current work before implementation starts.

1. Identify what's being built (from conversation context or current task)
2. Run `/guard classify` on the proposed change
3. If R0 — "No gate. Proceed."
4. If R1/R2 — search for a matching spec:
   - Check `spec/designs/` for recent specs matching the topic
   - Check conversation history for an approved design
5. Report:
   - **GATED** — spec found, approved, proceed with implementation
   - **BLOCKED** — no spec found. Run `design write` before coding.
   - **STALE** — spec exists but is >7 days old or assumptions have shifted. Review before proceeding.

### `design review`

Run dissent on an existing spec before implementation begins.

1. Read the spec document
2. Run `/dissent review` against the design decisions
3. If dissent raises concerns, present them before approving the spec
4. If no concerns, mark the spec as reviewed

---

## Proactive Behavior

This skill should auto-fire when:

- Code is about to be written for an R1/R2 change and no spec exists
- A plan is being discussed without a written spec
- Implementation starts with "let's just build it" energy on non-trivial work

**How to detect:** If the conversation moves toward implementation (file edits, code generation) and `/guard classify` would tag the work as R1/R2, check for a spec first.

---

## Rules

- **Three sentences minimum for R1.** Not zero, not a novel. Three sentences that capture what, why, and what it touches.
- **R0 is always exempt.** Never slow down trivial changes with design gates.
- **The spec lives in a file, not in conversation.** Persistent, reviewable, dispatchable to other terminals.
- **Stale specs are worse than no specs.** A spec written 3 weeks ago with shifted assumptions gives false confidence. Flag staleness.
- **Design gate is a service, not bureaucracy.** If the operator says "skip the gate," respect the override — but log it.

---

## Optional Integration

- Before `design write` → `/guard classify` to confirm reversibility level
- After spec is written → `/dissent review` to stress-test the design
- After spec is approved → `/breakdown` to create a task plan
- If spec touches other repos → `/handoff write` to notify
- If spec establishes new invariants → `/spec bind` to ratify

These are recommendations, not requirements. This skill works fully standalone.
