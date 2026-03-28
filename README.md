# kanly

> *"Kanly — the rules of vendetta under the Great Convention. The formal feud or war of assassins between Great Houses."*

Multi-repo governance toolkit for Claude Code. Twelve protocols governing how parallel sessions coordinate, what gets built, what doesn't, and when to push back — like the Great Convention governs how Houses interact.

## The Twelve Protocols

| Skill | Dune Analog | What it does |
|---|---|---|
| `/handoff` | **Dispatches** | Pass context between repos/sessions. What changed, what the other house needs to do. Verify API contracts against code. |
| `/spec` | **Treaties** | Manage SPEC.md with BINDING (ratified treaties) vs NON-BINDING (proposals). Approval gates on BINDING changes. |
| `/scope` | **Exclusion zones** | Manage NON_GOALS.md. Declare what's out of scope. Check if current work drifts into forbidden territory. |
| `/learn` | **The Spice** | Capture engineering learnings, gotchas, and migration patterns. Knowledge that must flow between sessions. |
| `/guard` | **The Gom Jabbar** | Classify change reversibility (R0/R1/R2). Map blast radius. Verify dead code. Enforce safety gates before the point of no return. |
| `/dissent` | **Truthsayer** | Surgical dissent — trace decision trees, self-answer from codebase, surface only what code can't resolve. |
| `/replace` | **Kanly itself** | Kill-and-prove protocol for full replacements. The old path must become provably dead. |
| `/design-gate` | **The Litany** | No spec, no code. Hard gate on R1/R2 implementation without a design. Forces the decision to be conscious. |
| `/breakdown` | **Battle Plans** | File-level task breakdowns for R1/R2 changes. Concrete enough for another agent to execute without questions. |
| `/review` | **The Council** | Cross-terminal spec compliance and code quality review. The agent that built it shouldn't be the only one who checks it. |
| `/test-gate` | **The Trial** | Tests required before R1 changes are marked complete. Not TDD — just proof the change is testable and tested. |
| `/verify` | **The Proof** | No completion claims without fresh verification evidence. Run the command, show the output, then claim success. |

## Install

```bash
# All twelve protocols
for skill in handoff spec scope learn guard dissent replace design-gate breakdown review test-gate verify; do
  cp -r kanly/$skill ~/.claude/skills/$skill
done

# Or symlink for development
for skill in handoff spec scope learn guard dissent replace design-gate breakdown review test-gate verify; do
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
/dissent                       # Surgical review — trace decisions, self-answer, surface gaps
/dissent review                # Same — explicit form
/dissent override              # Acknowledge concern, proceed anyway
/dissent log                   # Show dissents raised this session

# ── Design Gate: spec before code ──
/design write                  # Create a spec for the current work
/design check                  # Verify a spec exists before implementation
/design review                 # Run dissent on the spec before building

# ── Breakdown: task plans ──
/breakdown write               # Create file-level task breakdown
/breakdown view                # Show current plan with completion status
/breakdown validate            # Check for placeholders and missing dependencies

# ── Review: cross-terminal checks ──
/review write                  # Create a review dispatch after R1 change ships
/review check                  # Read and execute pending review dispatches
/review done                   # Archive a completed review

# ── Test Gate: coverage before done ──
/test check                    # Find tests for changed files
/test run                      # Run relevant tests, show output
/test gate                     # Final gate — coverage + passing required for R1

# ── Verify: evidence before claims ──
/verify run <command>          # Run command, capture output as evidence
/verify status                 # Check if current work has been verified
```

## Reversibility Classes

| Class | Meaning | Protocol |
|---|---|---|
| **R0** | Fully reversible | Move fast. No gate. |
| **R1** | Costly to reverse | Note it. Mention in commit. |
| **R2** | Hard to reverse | **STOP.** List failure modes. Ask approval. |

R2 triggers: schemas, money flows, contract deploys, public API changes, auth flow changes.

## The v2 Pipeline

New in v0.3.0 — five skills that enforce planning discipline and evidence-based completion. All gated by reversibility class. R0 stays fast. R1/R2 get the friction they deserve.

```
         /design-gate ← spec must exist for R1/R2
              │
              ▼
         /breakdown ← file-level task plan
              │
              ▼
         implement task by task
              │
              ▼
         /test-gate ← tests cover changed behavior
              │
              ▼
         /verify ← run tests, show output
              │
              ▼
         /review write ← dispatch for cross-terminal check
```

## Proactive Behavior (built-in)

Kanly skills auto-fire at the right moments — no CLAUDE.md snippet required:

| Skill | Auto-fires when... |
|---|---|
| `/guard` | Plan mode (classify changes), before commits (scan for R2), before changing exports (trace), after deletions (verify) |
| `/dissent` | Before finalizing any plan (surgical review — reads codebase first, surfaces only unresolvable questions) |
| `/replace` | When message starts with `REPLACE:` or intent is full replacement |
| `/spec` | During plan review (check BINDING compliance) |
| `/scope` | During planning (check for scope drift) |
| `/learn` | Starting work in a domain (surface gotchas) |
| `/handoff` | Session start (check dispatches), after cross-repo changes |
| `/design-gate` | Before R1/R2 implementation starts (check spec exists) |
| `/breakdown` | After design spec approved, before R1/R2 implementation |
| `/test-gate` | Before R1 changes are marked complete |
| `/verify` | Before any completion claim |
| `/review` | After R1 changes ship (suggest review dispatch) |

Want to customize triggers? See `CLAUDE_SNIPPET.md` for optional CLAUDE.md overrides.

## Philosophy

- **Plain markdown.** No database, no API. Just files.
- **Human-readable.** Useful even without Claude.
- **Convention over configuration.** Standard paths, standard format.
- **Async coordination.** Sessions don't need to be running simultaneously.
- **Governance, not bureaucracy.** The point is intentionality, not paperwork.
- **Zero config.** Skills self-trigger — no CLAUDE.md wiring needed.
- **Reversibility-aware.** R0 stays fast. Friction scales with blast radius.
- **Self-answering.** Agents check the codebase before asking the operator. Fewer questions, higher signal.

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
    designs/                   # Design specs from /design-gate
  docs/
    LEARNINGS.md               # Engineering learnings by domain
```

## How They Work Together

```
         /design-gate ← spec must exist
              │
         /dissent ← surgical review of the spec
              │
         /breakdown ← file-level task plan
              │
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
      ▼       ▼
   /test-gate ← tests cover changed behavior
      │
   /verify ← run, show output, prove it works
      │
   ┌──┴───────┬──────────┐
   │          │          │
/learn add /review write /handoff write
   │          │          │
   │     /scope check    │
   └──────────┴──────────┘
```

## License

MIT
