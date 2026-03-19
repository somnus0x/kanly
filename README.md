# kanly

> *"Kanly — the rules of vendetta under the Great Convention. The formal feud or war of assassins between Great Houses."*

Multi-repo governance toolkit for Claude Code. Seven protocols governing how parallel sessions coordinate, what gets built, what doesn't, and when to push back — like the Great Convention governs how Houses interact.

## The Seven Protocols

| Skill | Dune Analog | What it does |
|---|---|---|
| `/handoff` | **Dispatches** | Pass context between repos/sessions. What changed, what the other house needs to do. Verify API contracts against code. |
| `/spec` | **Treaties** | Manage SPEC.md with BINDING (ratified treaties) vs NON-BINDING (proposals). Approval gates on BINDING changes. |
| `/scope` | **Exclusion zones** | Manage NON_GOALS.md. Declare what's out of scope. Check if current work drifts into forbidden territory. |
| `/learn` | **The Spice** | Capture engineering learnings, gotchas, and migration patterns. Knowledge that must flow between sessions. |
| `/guard` | **The Gom Jabbar** | Classify change reversibility (R0/R1/R2). Map blast radius. Verify dead code. Enforce safety gates before the point of no return. |
| `/dissent` | **Truthsayer** | Raise concerns when a change increases coupling, hardens assumptions, or expands blast radius. |
| `/replace` | **Kanly itself** | Kill-and-prove protocol for full replacements. The old path must become provably dead. |

## Install

```bash
# All seven protocols
for skill in handoff spec scope learn guard dissent replace; do
  cp -r kanly/$skill ~/.claude/skills/$skill
done

# Or symlink for development
for skill in handoff spec scope learn guard dissent replace; do
  ln -s $(pwd)/kanly/$skill ~/.claude/skills/$skill
done
```

## Usage

```bash
# ── Handoff: dispatches ──
/handoff write frontend        # Write a dispatch (solo: repo, team: person)
/handoff check                 # Read pending dispatches
/handoff done frontend         # Delete after executing (git history = archive)
/handoff verify                # Check code matches the handoff's API contract

# ── Spec: BINDING/NON-BINDING treaties ──
/spec bind "Settlement must complete within 24h of resolution"
/spec note "Consider batch settlement for gas efficiency"
/spec check                    # Verify code against BINDING specs
/spec diff                     # What changed in spec since last commit

# ── Scope: non-goals exclusion zones ──
/scope add "Mobile native app — web-only for now"
/scope check                   # Detect scope drift in current work
/scope list                    # List all declared non-goals

# ── Learn: engineering learnings ──
/learn add "MiniEVM requires gas fees in EVM token denomination, not uinit"
/learn check auth              # Check gotchas before working in a domain
/learn list                    # All learnings by domain

# ── Guard: reversibility gates ──
/guard classify                # Tag current change as R0/R1/R2
/guard check                   # Scan staged changes for R2 territory
/guard tripwire                # Show all R2 domains for this project
/guard trace <symbol>          # Map blast radius before changing something
/guard verify <symbol>         # Verify removed code is actually dead

# ── Replace: kill-and-prove ──
/replace                       # Full replacement — old path must die
                               # Orchestrates: confirm → trace → execute → verify

# ── Dissent: the Truthsayer ──
/dissent                       # Review current approach for concerns
/dissent review                # Same — explicit form
/dissent override              # Acknowledge concern, proceed anyway
/dissent log                   # Show dissents raised this session
```

## Reversibility Classes

| Class | Meaning | Protocol |
|---|---|---|
| **R0** | Fully reversible | Move fast. No gate. |
| **R1** | Costly to reverse | Note it. Mention in commit. |
| **R2** | Hard to reverse | **STOP.** List failure modes. Ask approval. |

R2 triggers: schemas, money flows, contract deploys, public API changes, auth flow changes.

## Proactive Behavior (built-in)

Kanly skills auto-fire at the right moments — no CLAUDE.md snippet required:

| Skill | Auto-fires when... |
|---|---|
| `/guard` | Plan mode (classify changes), before commits (scan for R2), before changing exports (trace), after deletions (verify) |
| `/dissent` | Before finalizing any plan (review for risks) |
| `/replace` | When message starts with `REPLACE:` or intent is full replacement |
| `/spec` | During plan review (check BINDING compliance) |
| `/scope` | During planning (check for scope drift) |
| `/learn` | Starting work in a domain (surface gotchas) |
| `/handoff` | Session start (check dispatches), after cross-repo changes |

Want to customize triggers? See `CLAUDE_SNIPPET.md` for optional CLAUDE.md overrides.

## Philosophy

- **Plain markdown.** No database, no API. Just files.
- **Human-readable.** Useful even without Claude.
- **Convention over configuration.** Standard paths, standard format.
- **Async coordination.** Sessions don't need to be running simultaneously.
- **Governance, not bureaucracy.** The point is intentionality, not paperwork.
- **Zero config.** Skills self-trigger — no CLAUDE.md wiring needed.

## File Layout

```
# ── Solo mode (personal cross-repo) ──
~/.claude/handoffs/
  frontend.md                  # You → your frontend terminal
  backend.md                   # You → your backend terminal

# ── Team mode (in-repo, committed to git) ──
<repo>/.claude/handoffs/
  isagi--frontend-team.md      # isagi → frontend team
  alice--bob.md                # alice → bob
  bob--all.md                  # bob → everyone

# ── Shared project files ──
<repo>/
  spec/
    SPEC.md                    # BINDING and NON-BINDING treaties
    NON_GOALS.md               # Exclusion zones
  docs/
    LEARNINGS.md               # Engineering learnings by domain
```

## How They Work Together

```
              /dissent ← review before R1/R2 commits
                  │
                  ▼
            /guard classify
                  │
             R0? R1? R2?
                  │
       ┌──────────┼──────────┐
       │          │          │
    R0: go     R1: note   R2: STOP
                  │          │
                  │    List failure modes
                  │    Ask approval
                  │          │
                  ▼          ▼
           /guard trace ← map blast radius
                  │
               Make the change
                  │
          ┌───────┤ (if replacement)
          │       │
   /guard verify  │
   (confirm kill) │
          │       │
       ┌──┴───────┼──────────┐
       │          │          │
  /learn add  /spec bind  /handoff write
       │          │          │
       │     /scope check    │
       └──────────┴──────────┘
```

## License

MIT
