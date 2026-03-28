---
name: verify
description: Enforce evidence-based completion claims. Use before claiming any work is done, fixed, passing, or complete. Use when you detect phrases like "should work now", "looks correct", "I'm confident", or any completion claim without fresh command output in the same message. Use proactively before commits, PRs, or status reports.
argument-hint: [run <command>|status]
---

# Verify — Evidence Before Assertions

No completion claims without fresh verification evidence. Run the command, show the output, then claim success.

---

## What This Skill Does

"Should work now" is not verification. "Looks correct" is not verification. "I'm confident" is not verification. This skill enforces a single rule: if you claim something works, show the output that proves it. In the same message. Fresh.

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

---

## Commands

### `verify run <command>`

Run a command and capture its output as evidence.

1. Execute the full command (not a subset, not a partial run)
2. Capture stdout, stderr, and exit code
3. Present the output
4. Only THEN make a status claim based on what the output shows

**Output format:**
```
VERIFY: <command>
Exit: <0|non-zero>

<actual output>

Status: <PASS|FAIL|ERROR> — <one-line summary of what the output shows>
```

### `verify status`

Check if the current work has been verified.

1. Review the current session — was a verification command run after the last code change?
2. Report:
   - **VERIFIED** — fresh evidence exists in this message/session
   - **UNVERIFIED** — code was changed but no verification command was run since
   - **STALE** — verification was run, but code changed after. Re-verify.

---

## Red Flags — STOP

If any of these appear in a draft response, verification is missing:

| Red flag | What to do |
|---|---|
| "Should work now" | Run the verification command |
| "Looks correct" | Run the verification command |
| "I'm confident" | Confidence ≠ evidence |
| "I edited the file" | Show the output, not just the edit |
| "Done" / "Fixed" / "Shipped" | Where's the proof? |
| "Probably passes" | Run it and find out |
| Expressing satisfaction before output | Delete the satisfaction, run the command |

## What Counts as Verification

| Claim | Requires | Not sufficient |
|---|---|---|
| Tests pass | Test command output showing 0 failures | Previous run, "should pass" |
| Build succeeds | Build command with exit 0 | "Linter passed" |
| Bug fixed | Reproduce original symptom → now passes | "Code changed, assumed fixed" |
| Cron job works | Run it, show the output | "I edited the crontab" |
| File updated | Show the relevant content after edit | "I edited the file" |
| Config change applied | Restart/reload + verify behavior | "I changed the config" |

---

## Rules

- **Fresh means fresh.** Output from a previous session doesn't count. Run it now.
- **Full means full.** Don't run a subset of tests and claim "all tests pass."
- **Exit code matters.** Check it. A command that prints errors but exits 0 is still suspicious.
- **Verification is the last step, not an afterthought.** Build it into the workflow, not as a "oh right, should I check?"
- **Override is fine.** If the operator says "trust me, move on" — respect it. But the default is verify.

---

## Optional Integration

- After `/test-gate` confirms tests exist → `verify run` to prove they pass
- After `/review` checks spec compliance → `verify run` on the test suite
- Before claiming a `/replace` is complete → `verify` the old path is dead
- After any `/guard` R2 change lands → `verify` the behavior is correct

This skill is referenced by most other kanly skills as the final completion step.
