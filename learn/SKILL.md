---
name: learn
description: Manage engineering learnings, gotchas, and patterns per project. Use when user says "learn", "learning", "gotcha", or after discovering a non-obvious fix, migration pattern, or debugging insight worth preserving.
argument-hint: [add <insight>|check <domain>|list]
---

# Learn — The Spice Must Flow

Capture and retrieve engineering learnings so hard-won knowledge isn't lost between sessions. Every painful debug, every migration gotcha, every "oh THAT's how it works" — preserved.

**File:** `<project-root>/docs/LEARNINGS.md`

---

## Commands

### `learn add <insight>`

Capture a learning after discovering something non-obvious.

1. Read current `docs/LEARNINGS.md` (create with header if it doesn't exist)
2. Determine the domain category (see categories below)
3. Add the learning under the appropriate `## <Domain>` section
4. Use the format below
5. If a section for that domain doesn't exist, create it

**Format:**

```markdown
### <Short descriptive title>

<What happened or what we discovered — 1-3 sentences>

<Code snippet, config, or command if relevant>

Key takeaway: <one-line summary of what to remember>
```

**What qualifies as a learning:**
- Non-obvious behavior that caused a bug or confusion
- Migration patterns (old way → new way)
- Platform/framework gotchas with workarounds
- Configuration that isn't documented well upstream
- Debugging insights ("if you see X, it's because Y")

**What does NOT qualify:**
- Standard API usage (that's what docs are for)
- Personal preferences or style choices
- Temporary workarounds (use TODO comments instead)

### `learn check <domain>`

Before working in a domain, check if there are known gotchas.

1. Read `docs/LEARNINGS.md`
2. Find all entries matching the domain keyword (fuzzy match — "auth" matches "Auth", "Authentication", "Privy Auth")
3. Summarize relevant learnings as warnings
4. If nothing matches, say "No known gotchas for <domain>."

Use this proactively when:
- Starting work in a domain you haven't touched recently
- Debugging something that "should work but doesn't"
- Before a migration or refactor

### `learn list`

List all learnings organized by domain.

1. Read `docs/LEARNINGS.md`
2. Print each domain section with a count of entries
3. Show last modified date

---

## Domain Categories

Organize learnings under these (or add new ones as needed):

- **Blockchain** — chain-specific behavior, gas, transactions, token decimals
- **Auth** — authentication flows, token handling, provider quirks
- **Database** — Firestore/SQL gotchas, transaction patterns, query limitations
- **API** — endpoint behavior, error handling, serialization
- **Frontend** — rendering, state management, browser quirks
- **DevOps** — deployment, CI/CD, Docker, environment config
- **Testing** — test setup gotchas, mocking patterns, flaky test fixes
- **Migration** — old → new patterns, deprecation notes

---

## File Template

When creating `docs/LEARNINGS.md` for the first time:

```markdown
# <Project Name> — Engineering Learnings

Consolidated reference of key decisions, fixes, and patterns learned during development.

---

## <First Domain>

### <First Learning Title>

<Description>

Key takeaway: <summary>
```

---

## Rules

- **Learnings are permanent** — don't delete them unless they're provably wrong
- **One learning per gotcha** — keep entries focused and scannable
- **Code snippets welcome** — show the fix, not just describe it
- **Key takeaway required** — every entry ends with the one thing to remember
- **Check before you dig** — run `learn check <domain>` before debugging in unfamiliar territory
