---
name: replace
description: Full replacement protocol — old behavior must become unreachable. Use when user says "replace", message starts with "REPLACE:", or intent is to swap out an old implementation entirely (not extend, not add alongside). Use when user says "swap out", "remove the old", "kill the old path", "replace X with Y". Do NOT use for additive changes or feature additions. The key distinction is REPLACEMENT (old becomes dead) vs ADDITION (old stays alive).
argument-hint: [<description of what to replace>]
---

# Replace — Kill-and-Prove Protocol

Full replacement of old behavior. The old path must become unreachable. Not "deprecated," not "unused" — provably dead.

---

## What This Skill Does

The most dangerous refactor is the one that leaves old code alive alongside new code. Callers split between paths. Bugs hide in the old path nobody tests anymore. State diverges. This skill ensures that when you replace something, the old version is actually gone.

## When to Use

- Swapping an old implementation for a new one
- Migrating from one data source to another
- Replacing an endpoint, handler, or service
- Moving from one library/SDK to another

## When NOT to Use

- Adding a feature alongside existing code → normal change
- A/B testing or feature flags → old path intentionally stays alive
- Extending an interface → additive, not replacement

---

## Protocol

### Step 1: Confirm intent

Ask one confirmation:

```
Confirm: FULL replacement — old behavior becomes unreachable. Yes/No?
```

If No → treat as a normal change. Stop this protocol.

### Step 2: Map the blast radius

Run `/guard trace` on the thing being replaced. Identify:

1. **All old entrypoints** — routes, functions, handlers, exports, callsites
2. **All consumers** — who calls the old thing, imports it, references it
3. **Cross-repo exposure** — is this used outside this repo? If yes, `/handoff write` is required.

Output:

```
REPLACE MAP: <old thing> → <new thing>

Old entrypoints (N):
  - <file:line> — <description>

Consumers (N):
  - <file:line> — <how it uses the old thing>

Cross-repo: <yes/no>
```

### Step 3: Choose kill mechanism

Pick exactly ONE mechanism. Don't mix approaches.

| Mechanism | When to use | Example |
|---|---|---|
| **Delete** | Old code has no external consumers | Remove the file/function entirely |
| **Throw** | Old path might be hit by accident | `throw new Error('REMOVED: use newThing instead')` |
| **Rename** | Need a transition period | `oldThing_OLD_DO_NOT_USE()` |
| **Single switch** | Multiple callsites, same interface | Swap the import/binding in one place |

**Prefer delete.** The other mechanisms are fallbacks for when you can't prove all callers are updated.

### Step 4: Execute replacement

1. Write the new implementation
2. Migrate all consumers from old → new
3. Apply the kill mechanism to the old path
4. Update all tests to use the new path

### Step 5: Prove the kill

Run `/guard verify` on the old symbol. The result MUST be **DEAD** or **GHOST**.

If **ALIVE** → go back to Step 4 and fix remaining references.

Additionally, add at least ONE of these proofs:

| Proof type | What it does |
|---|---|
| **Test** | A test that asserts the old path throws or doesn't exist |
| **Grep** | Show `grep` output proving zero references to old symbol |
| **Runtime** | A log/throw in the old path — if it ever fires, the replacement failed |

### Step 6: Report

```
REPLACED: <old thing> → <new thing>

Kill mechanism: <delete|throw|rename|switch>
Consumers migrated: N
Proof: <test|grep|runtime> — <one-line description>
Cross-repo handoff: <written|not needed>

Guard verify: DEAD ✓
```

---

## Migration Map Template

For data source migrations (e.g., Firestore → API, old endpoint → new endpoint):

```
MIGRATION MAP:

| Consumer | Old source | New source | Status |
|---|---|---|---|
| Home feed | Firestore onSnapshot | GET /bets?limit=5 | ✓ migrated |
| Profile page | Firestore query | GET /bets/me | ✓ migrated |
| Footer ticker | Firestore onSnapshot | GET /bets?limit=10 (polled) | ✗ pending |

Remaining: N consumers not yet migrated
```

Do NOT mark the replacement as complete until all rows show ✓.

---

## Optional Integration

If other kanly skills are installed:
- **Before Step 2** → `/guard trace` to map blast radius (required by this protocol)
- **After Step 5** → `/guard verify` to confirm kill (required by this protocol)
- **If cross-repo** → `/handoff write` to notify consumers (required if cross-repo exposure detected)
- **If replacement reveals a gotcha** → `/learn add`
- **If replacement changes a spec boundary** → `/spec note`

`/guard trace` and `/guard verify` are integral to this protocol. If guard is not installed, perform the same steps manually (grep for references, verify no survivors).

---

## Rules

- **One kill mechanism per replacement.** Don't delete in some places and throw in others.
- **Never mark complete with ALIVE references.** Fix them or acknowledge the survivors explicitly.
- **Delete > Throw > Rename.** Prefer the most aggressive kill mechanism you can safely use.
- **Cross-repo replacements require handoffs.** You can't kill something that lives in another repo without telling them.
- **No "soft deprecation."** If you're not willing to kill the old path, this isn't a REPLACE: — it's an addition. Use normal workflow.
- **Migration maps must be exhaustive.** Every consumer gets a row. No "and others."
