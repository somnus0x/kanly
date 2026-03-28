# kanly — Optional CLAUDE.md Integration

> Proactive behavior is now **built into the skill descriptions** — kanly skills auto-fire
> during plan mode, before commits, at session start, etc. without any CLAUDE.md changes.
>
> This snippet is OPTIONAL — only paste it if you want to **customize** the proactive triggers
> or add project-specific overrides beyond the defaults.

```markdown
# Kanly Governance Hooks (optional overrides)

## Proactive Checks (customize defaults)

Before R1/R2 implementation:
- Run `/design check` to verify a spec exists. If not, run `/design write` before touching code.
- Run `/breakdown write` to create a file-level task plan.

Before non-trivial changes:
- Run `/guard classify` to tag reversibility. If R1/R2, run `/dissent review` on the approach.
- Run `/guard trace` to map blast radius before changing any exported symbol, endpoint, or type.

Before working in a domain:
- Check `docs/LEARNINGS.md` for known gotchas matching the active domain.

Before marking R1 changes complete:
- Run `/test gate` to verify test coverage exists and passes.
- Run `/verify run` to prove the tests pass with fresh output.

After R1 changes ship:
- Run `/review write` to create a cross-terminal review dispatch.

After changes that affect other repos or people:
- Write a `/handoff` dispatch. Delete it after the recipient has executed on it.

After receiving a handoff:
- Run `/handoff verify` to check if your code matches the declared API contract.

After removing or replacing code:
- Run `/guard verify` to confirm the old path is dead.

When replacing old behavior entirely:
- Use `/replace` to enforce the kill-and-prove protocol.

Before any completion claim:
- Run `/verify run` with the relevant command. No "should work now" — evidence only.

## Skills Reference

- `/handoff` — cross-repo dispatches (write, check, done, verify)
- `/spec` — BINDING/NON-BINDING spec management (bind, note, check, diff)
- `/scope` — non-goals exclusion zones (add, check, list)
- `/learn` — engineering learnings & gotchas (add, check, list)
- `/guard` — R0/R1/R2 reversibility gates (classify, check, tripwire, trace, verify)
- `/dissent` — surgical disagreement with decision-tree traversal (review, override, log)
- `/replace` — kill-and-prove replacement protocol
- `/design-gate` — spec-before-code gate for R1/R2 (write, check, review)
- `/breakdown` — file-level task plans for R1/R2 (write, view, validate)
- `/review` — cross-terminal spec compliance review (write, check, done)
- `/test-gate` — test coverage gate for R1 (check, run, gate)
- `/verify` — evidence-based completion (run, status)
```
