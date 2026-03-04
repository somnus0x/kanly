---
name: guard
description: Classify how reversible a code change is before applying it. Use before non-trivial changes, or when user says "guard", "classify", "reversibility", or when changes touch database schemas, money/payment logic, smart contracts, public APIs, or auth flows. These are high-risk domains where mistakes are hard to undo.
argument-hint: [classify|check|tripwire]
---

# Guard — Reversibility Gate

Test every change before it lands. Classify how reversible it is, enforce safety gates, and stop before the point of no return.

---

## What This Skill Does

Software changes vary in how easily they can be undone. A CSS tweak is trivial to revert. A database migration on production data is not. This skill classifies changes into three tiers and enforces appropriate caution for each.

## Reversibility Classes

| Class | Meaning | Examples | Protocol |
|---|---|---|---|
| **R0** | Fully reversible | Rename variable, CSS tweak, add logging, refactor internal function | Move fast. No gate needed. |
| **R1** | Costly to reverse | Config change, dependency upgrade, feature flag toggle, new DB index | Note it in the commit message. |
| **R2** | Hard or impossible to reverse | Schema migration, money flow change, contract deploy, public API shape change, auth flow change | **STOP.** List what goes wrong if this is wrong. Get explicit approval before proceeding. |

---

## Commands

### `guard classify`

Classify the current change's reversibility before applying it.

1. Look at what's about to change (staged files, current diff, or described change)
2. Check each file/change against the R2 tripwire domains (see below)
3. Assign R0, R1, or R2 with reasoning
4. Output the classification tag

**Output format:**
```
**R<N>** (<category>) — <one-line reason>
```

Examples:
```
**R0** (ui) — CSS color change, fully reversible
**R1** (dependency) — Upgrading viem from 2.33 to 2.34, rollback possible but costly
**R2** (schema) — Adding required field to user documents, existing records won't have it
```

### `guard check`

Scan current changes for R2 territory. Run before committing.

1. Check `git diff --staged` (or `git diff` if nothing staged)
2. For each changed file, check against tripwire domains
3. Report:
   - **Clear** — no R2 territory detected
   - **Warning** — R1 changes detected, listed with reasoning
   - **R2 STOP** — hard-to-reverse changes detected. List each with:
     - What's changing
     - Why it's R2
     - Failure modes (what goes wrong if this is wrong)
     - Minimal safe approach (if any)
4. If R2 detected, **do not proceed without explicit user approval**

### `guard tripwire`

Show the R2 tripwire domains for this project.

1. List all default R2 domains (see below)
2. If `spec/R2_DOMAINS.md` exists, merge project-specific domains
3. Check if any are relevant to recent work (last 5 commits)
4. Flag any that have been touched without R2 classification

---

## R2 Tripwire Domains

These domains are **always R2** — any change here triggers the full safety gate:

### Money & Value
- Token amounts, decimal handling, unit conversions
- Deposit/withdrawal flows, balance calculations
- Fee structures, pricing logic, settlement/payout logic

### Data & Schema
- Database schema changes (new required fields, renamed/deleted fields)
- Collection/table structure changes, index changes on production data
- Data migration scripts

### Contracts & Chain
- Smart contract storage layout, event signatures, function signatures
- Chain configuration (RPC endpoints, chain IDs)

### Public Interfaces
- REST API endpoints (path, method, request/response shape)
- WebSocket/event schemas, SDK/library public exports, webhook formats

### Auth & Security
- Authentication flow changes, token validation logic
- Permission/role changes, encryption/hashing changes

### Infrastructure
- Database connection config, CI/CD pipeline changes
- Docker base image changes, resource limits, health check paths

### Project-Specific Domains
If `spec/R2_DOMAINS.md` exists in the project root, read it and merge with the defaults above. This lets each project define additional R2 domains specific to its domain.

---

## Optional Integration

If other kanly skills are installed, these work well together:
- After R2 is flagged → `/spec bind` if a new spec rule is needed
- After a non-obvious fix → `/learn add` to capture the gotcha
- If R2 change affects another repo/person → `/handoff write`

These are recommendations, not requirements. This skill works fully standalone.

---

## Rules

- **R2 is a conversation, not a blocker** — the point is intentional decisions, not preventing changes
- **Classify early** — tag reversibility when planning, not after coding
- **Failure modes required for R2** — "what goes wrong if this is wrong?" must be answered
- **Minimal safe approach** — always propose the smallest version of an R2 change
- **No silent R2** — if you notice an R2 change that wasn't classified, flag it immediately
