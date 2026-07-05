---
name: contract
description: Declare named, testable exit assertions BEFORE an autonomous run, then gate completion on all of them passing. Use before any goal-mode / mission-mode / loop-until-green task, when the user says "contract", "define done", "what's the exit condition", or whenever an agent is about to run autonomously toward a goal. Use proactively before handing a task to a self-looping harness (Codex goal mode, Factory mission mode, Claude dynamic workflow) so the loop has a real finish line, not a vibe.
argument-hint: [declare <task>|check|close]
---

# Contract — The Mentat's Binding

Before an autonomous loop runs, declare what "done" means as named, testable assertions. The loop is not complete until every assertion passes. No assertion, no autonomous run.

---

## What This Skill Does

`spec` governs the *project* (treaties that outlive any one task). `contract` governs a *single task* (what done means for THIS run, this session, this goal). The distinction matters: an autonomous loop with no declared exit condition runs until it *feels* done, drifts, or hits a timeout — none of which is "done."

This skill forces the exit condition to exist, be named, and be testable BEFORE the loop starts. The pattern is a validation-contract: name the assertions first (VAL-1, VAL-2…), build second, validate against the named set last.

**Receipt:** on a recent build, mission mode declared 15 named assertions before writing any implementation, then a separate validator pass gated completion on all 15. The loop didn't self-declare done; the contract did.

## When to Use

- Before any goal-mode / mission-mode / loop-until-green run
- Before handing a task to a self-looping harness (Codex, Factory, Claude ultracode)
- When a task will run autonomously for more than a few steps
- When "done" is fuzzy and the loop could plausibly stop early or run forever

## When NOT to Use

- A single deterministic edit (no loop, no autonomy) → just `verify` the output
- Project-level invariants → that's `spec bind`, not a task contract
- Exploration with no defined endpoint → contracts gate *completion*, not discovery

---

## The Iron Law

```
NO AUTONOMOUS LOOP WITHOUT A GREEN-ABLE EXIT CONTRACT
```

If you can't write the assertions that mean "done," the task isn't ready to run autonomously. Stop and define them first.

---

## Commands

### `contract declare <task>`

Define the exit contract for an autonomous task before it runs.

1. Restate the task in one line
2. Decompose "done" into named, testable assertions — each one independently verifiable
3. For each assertion, state HOW it gets checked (the command, the grep, the observable behavior)
4. Write the contract to `contract/CONTRACT-<task-slug>.md`
5. The loop may now run, bound to this contract

**Assertions must be:**
- **Named** — `CONTRACT-1`, `CONTRACT-2`… so completion can be reported per-assertion
- **Testable** — a command, grep, or observable behavior decides pass/fail (no "looks good")
- **Binary** — passes or fails, no partial credit
- **Pre-declared** — written BEFORE the build, not reverse-engineered to match what got built

**Contract format:**
```markdown
# Contract: <task>

Declared: <date> — before build
Bound harness: <Codex goal | Factory mission | Claude ultracode | manual>

## Exit Assertions

### CONTRACT-1: <what must be true>
Check: <command | grep | observable behavior>
<!-- why: <why this is part of done> -->

### CONTRACT-2: <what must be true>
Check: <command | grep | observable behavior>

## Completion gate
Done = ALL assertions pass. Any FAIL = loop continues or task reopens.
```

### `contract check`

Validate the current work against the active contract. This is the gate.

1. Read the active `contract/CONTRACT-<slug>.md`
2. For each assertion, run its declared check and capture fresh output (defers to `verify run`)
3. Report per-assertion:

```
CONTRACT CHECK: <task>

CONTRACT-1: PASS — <evidence one-liner>
CONTRACT-2: FAIL — <what the check showed>
...

Gate: <N/M passing> — <COMPLETE | INCOMPLETE>
```

4. If any FAIL → the task is INCOMPLETE. Do not claim done. The loop continues.
5. Only ALL-PASS lets the task close.

### `contract close`

Close a satisfied contract.

1. Run `contract check` — must be ALL-PASS (no closing on partial)
2. Record the closing evidence in the contract file
3. Optionally archive: contract is satisfied, git history is the record
4. If the contract revealed a reusable invariant → suggest `spec bind` to promote it to project law

---

## Why Pre-Declaration Matters

The whole value is in declaring assertions BEFORE the build. Reverse-engineered contracts (writing assertions to match what the loop already produced) are theater — they always pass, because they were fitted to the result. A real contract can FAIL its own build, which is the point: it's the only thing that catches a loop that confidently finished the wrong thing.

| Anti-pattern | What's wrong | Fix |
|---|---|---|
| Writing assertions after the build | Fitted to pass — proves nothing | Declare before the loop runs |
| "Done when it works" | Not testable, not binary | Name the observable that means "works" |
| One giant assertion | Can't report partial progress, hides which part failed | Split into independent named assertions |
| Timeout = done | A loop that idled to its cap didn't pass — it ran out of clock | Gate on assertions, never on time |

---

## Rules

- **Declare before build.** A contract written after the loop ran is not a contract, it's a description.
- **Named and binary.** Every assertion has an ID and a pass/fail check. No prose verdicts.
- **All-pass or incomplete.** No closing a contract with a FAIL. Partial is incomplete.
- **The contract gates the loop, not the clock.** A timed-out autonomous run is INCOMPLETE, not done.
- **Contracts are per-task and disposable.** Unlike `spec` (durable project law), a contract dies when the task closes. Promote durable findings to `spec bind`.
- **Override is fine.** Operator can close a contract manually with `dissent override` reasoning — but the default is all-pass.

---

## Optional Integration

If other kanly skills are installed:
- **Before declaring** → `/breakdown write` decomposes the task into files; contract decomposes "done" into assertions. Do both for R1.
- **Each `contract check`** → defers to `/verify run` for fresh per-assertion evidence (contract is the *set* of verifications, verify is *each one*).
- **On close, if a reusable invariant emerged** → `/spec bind` to promote task-level assertion to project-level law.
- **If an assertion can't be made testable** → that's a `/dissent` signal: the task's "done" is under-specified, surface it before running.
- **Bound to an autonomous harness** → the contract is what makes goal-mode / mission-mode loops have a real finish line. Declare the contract, then let the harness loop against it.

`contract` is the task-scoped complement to `spec`'s project scope and `verify`'s single-command proof. It governs the one thing neither does: the exit condition of an autonomous run, declared before the run starts.
