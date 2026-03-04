# kanly — Optional CLAUDE.md Integration

> Paste this into your project or global CLAUDE.md for tighter kanly integration.
> This is OPTIONAL — all kanly skills work standalone without it.
> This snippet adds proactive behavioral hooks that make skills fire automatically.

```markdown
# Kanly Governance Hooks

## Proactive Checks (auto-behavior)

Before non-trivial changes:
- Run `/guard classify` to tag reversibility. If R1/R2, run `/dissent review` on the approach.

Before working in a domain:
- Check `docs/LEARNINGS.md` for known gotchas matching the active domain.

After changes that affect other repos or people:
- Write a `/handoff` dispatch. Delete it after the recipient has executed on it.

## Skills Reference

- `/handoff` — cross-repo dispatches (write, check, done)
- `/spec` — BINDING/NON-BINDING spec management (bind, note, check, diff)
- `/scope` — non-goals exclusion zones (add, check, list)
- `/learn` — engineering learnings & gotchas (add, check, list)
- `/guard` — R0/R1/R2 reversibility gates (classify, check, tripwire)
- `/dissent` — structured disagreement (review, override, log)
```
