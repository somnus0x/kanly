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

## Surgical Dissent Process

When dissent is triggered (automatically or via command), follow this process:

### Step 1: Trace the decision tree

Identify every decision embedded in the proposed change. Not just "is this risky?" but "what specific decisions are being made here, and what does each one assume?"

For each decision branch:
- What's being decided?
- What alternatives exist?
- What assumption makes this the right choice?

### Step 2: Self-answer from codebase

**Before asking the operator anything**, check the codebase:

- Read the relevant files. Do they confirm or contradict the assumptions?
- Check test coverage. Is the changed behavior tested?
- Check for existing patterns. Does this change follow or break them?
- Check cross-repo surface. Does this affect other repos?

If the codebase answers the question, state what you found. Don't ask the operator to verify what you can verify yourself.

### Step 3: Surface only what code can't resolve

After self-answering, surface only the questions that require human judgment:
- Design intent ("why this approach over X?")
- Business logic ("does the product actually need this?")
- User impact ("how does this affect the onboarding flow?")
- Strategic tradeoffs ("is this the right time for this change?")

This reduces dissent fatigue. Generic questions get ignored. Surgical questions based on code analysis earn attention.

---

## Structured Debate Rounds

For R1 decisions (costly to reverse), dissent escalates from single-shot to a **structured bull/bear debate** with a hard round limit. Stolen from TradingAgents' adversarial debate pattern — but compressed for a solo builder, not a trading desk.

### When to trigger

Structured debate activates when:
- The change is R1 (costly to reverse) — database migrations, API contracts, architectural changes
- Multiple valid approaches exist and the tradeoffs aren't obvious
- The operator explicitly asks: `dissent debate`

Do NOT use structured debate for R2 decisions. Single-shot dissent is enough.

### How it works

**Round 1 — Bull case**
State the strongest argument FOR the proposed approach. Be genuine — steelman it.
- What problem does it solve?
- What does the operator gain?
- What's the best-case outcome?

**Round 1 — Bear case**
State the strongest argument AGAINST. No softening.
- What breaks or becomes harder?
- What assumption is most fragile?
- What's the worst-case outcome?

**Round 2 — Rebuttal**
Bull responds to the bear's strongest point. Bear responds to the bull's strongest point. One rebuttal each — no infinite loops.

**Judgment**
After 2 rounds, render a verdict:

```
DEBATE VERDICT: [PROCEED | PAUSE | REDESIGN]

Bull's strongest point: [1 sentence]
Bear's strongest point: [1 sentence]
Unresolved risk: [the thing neither side could fully answer]
Recommendation: [what to do next — proceed as-is, add a safeguard, or rethink]
```

### Hard limits

- **Max 2 rounds.** Not 3, not 5. Two rounds of bull/bear is enough to surface real risk. More rounds is deliberation theater.
- **Total debate output under 300 words.** Compress, don't expand.
- **No debate on R2 decisions.** If it's easily reversed, just do it.
- **Operator can cut the debate at any point** with `dissent override`. The debate serves the builder, not the process.

---

## Commands

### `dissent` (no args) or `dissent review`

Review the current approach for concerns using the surgical process above.

1. Trace the decision tree — identify every decision in the proposed change
2. Self-answer from codebase — read files, check tests, verify assumptions from code
3. Check against all five conditions (coupling, hardening, blast radius, scope creep, convention violation)
4. Surface only concerns that code can't resolve
5. If concerns found, raise using the format below
6. If no concerns, say "No dissent. Approach looks clean."

### `dissent debate`

Trigger structured bull/bear debate on the current approach (R1 decisions only).

1. Run Round 1 — Bull case (steelman the approach)
2. Run Round 1 — Bear case (strongest argument against)
3. Run Round 2 — One rebuttal each
4. Render verdict: PROCEED, PAUSE, or REDESIGN
5. Total output under 300 words

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
- `/design review` triggers dissent on a spec before implementation begins
- `/review check` can trigger dissent as part of cross-terminal review

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
