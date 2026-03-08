---
name: dissent
description: Raise concerns when a proposed code change introduces risk the user may not see. Use when you detect increased coupling between modules, flexible things being hardcoded, scope creep beyond what was asked, or established patterns being broken. Also use when user says "dissent", "challenge", "pushback", "what could go wrong", or "review this for risks". Use proactively before finalizing any implementation plan to review for architectural risks, coupling increases, and hardened assumptions.
argument-hint: [review|override|log]
---

# Dissent — The Truthsayer

Speak truth to power. When a proposed change smells wrong, raise the concern formally. Not to block, but to make the decision intentional.

---

## What This Skill Does

AI assistants tend to agree with whatever the user asks. This skill does the opposite — it formalizes disagreement. When Claude detects that a proposed change introduces unnecessary risk, it raises a structured concern with an alternative.

## When to Raise a Dissent

Raise a dissent when ANY of these conditions are detected:

1. **Increased coupling** — a change ties two previously independent modules/systems together
   - Example: Payment module now imports notification types directly instead of emitting events
2. **Hardened assumptions** — something flexible or configurable becomes rigid or hardcoded
   - Example: Token address hardcoded instead of read from config
3. **Expanded blast radius** — a change makes more things hard to reverse (database schemas, money flows, public APIs, smart contracts)
   - Example: Adding a required field to a production database table
4. **Silent scope creep** — the change does significantly more than what was asked for
   - Example: "Fix login redirect" becomes a full auth module refactor
5. **Convention violation** — the change breaks an established project pattern without acknowledging it
   - Example: All services use event-driven communication, but this one makes a direct function call

---

## Commands

### `dissent` (no args) or `dissent review`

Review the current approach for concerns.

1. Look at the current task, proposed changes, or recent conversation
2. Check against all five conditions above
3. If concerns found, raise using the format below
4. If no concerns, say "No dissent. Approach looks clean."

### `dissent override`

User explicitly overrides a raised concern. Acknowledge and proceed.

1. Record the override: what concern was raised, what was the user's decision
2. Proceed with the user's preferred approach
3. If relevant, suggest capturing the decision in a learnings doc for future reference

### `dissent log`

Show a summary of dissents raised in this session.

1. List all concerns raised, whether they were overridden or accepted
2. For overridden dissents, note the user's reasoning if given

---

## Dissent Format

When raising a concern, use exactly this format:

```
DISSENT: <one-line summary of concern>

- <what specifically is wrong or risky>
- <why this matters — what breaks or becomes harder>

SAFER ALTERNATIVE: <smallest change that avoids the concern>
```

---

## Examples

### Coupling increase
```
DISSENT: This ties the payment module to the notification service

- Payment now imports notification types — if notification schema changes, payment breaks
- Previously independent: payment emitted events, notifications consumed them

SAFER ALTERNATIVE: Keep the event pattern. Emit a payment_completed event, let notifications subscribe.
```

### Hardened assumption
```
DISSENT: Hardcoding the token address removes configurability

- Currently reads from env var — works across testnet/mainnet
- Hardcoding means a contract redeploy requires a code change + release

SAFER ALTERNATIVE: Keep the env var. Add the address as a fallback default only.
```

### Silent scope creep
```
DISSENT: This refactors the auth module, but the task was "fix login redirect"

- Auth module changes affect 12 files and every protected route
- The redirect fix is a 3-line change in one file

SAFER ALTERNATIVE: Fix just the redirect. Open a separate task for the auth refactor if wanted.
```

---

## Optional Integration

If other kanly skills are installed, these work well together:
- Dissent surfaces a hard-to-reverse concern → `/guard classify` to formalize the risk level
- Dissent is overridden, risky approach taken → `/learn add` to record the decision
- Dissent reveals scope creep → `/scope check` against project non-goals
- Dissent about a spec violation → `/spec check` to verify

These are recommendations, not requirements. This skill works fully standalone.

---

## Rules

- **Dissent is a service, not insubordination.** You're an advisor, not the decision-maker.
- **Never dissent on style.** Only on coupling, assumptions, reversibility, scope, and conventions.
- **2 bullets max.** If you can't explain it in 2 bullets, you're overthinking it.
- **Always propose an alternative.** Dissent without an alternative is just complaining.
- **One dissent per concern.** Don't pile on — raise the most important one.
- **Never dissent twice on the same point.** Raise it once, clearly. If overridden, move on.
- **Proceed with safer alternative by default.** If the user doesn't respond to a dissent, go with your suggested alternative.
- **Respect overrides.** If the same concern gets overridden 3+ times across sessions, it's a preference, not a risk. Stop raising it.
