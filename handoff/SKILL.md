---
name: handoff
description: Write or check cross-repo handoff notes between Claude Code sessions. Use when user says "handoff", "write handoff", "check handoffs", "verify contract", "api drift", "does this match the handoff", or after completing work that affects another repo. Use proactively at session start to check for pending dispatches (run `handoff check`). Use proactively after changes that affect API contracts, shared types, or cross-repo interfaces to prompt writing a dispatch. Use `verify` after receiving a handoff to check if your code matches the declared contract.
argument-hint: [write <target>|check|done <file>|verify]
---

# Handoff — Dispatches Between Houses

Pass context between repos/terminals through structured markdown notes.

---

## Mode Detection

Detect which mode to use automatically:

1. Check if `<repo-root>/.claude/handoffs/` exists → **Team mode**
2. Otherwise check if `~/.claude/handoffs/` exists → **Solo mode**
3. If neither exists, ask which to create.

| Mode | Location | Visibility | Archive |
|---|---|---|---|
| **Team** | `<repo>/.claude/handoffs/` | Committed to git, team sees on pull | Delete file + commit. Git history is the archive. |
| **Solo** | `~/.claude/handoffs/` | Your terminals only | Delete file. Gone. |

---

## Commands

### `handoff write <target>`

Write a dispatch. Target is a repo name (solo) or person/team (team mode).

**Solo mode:**
1. Write to `~/.claude/handoffs/<target>.md`

**Team mode:**
1. Determine author from git config: `git config user.name` → kebab-case (e.g., "isagi")
2. Write to `<repo>/.claude/handoffs/<author>--<target>.md`
   - `<author>` = who wrote it
   - `<target>` = who it's for (person, team, or "all")
   - Example: `isagi--frontend-team.md`, `alice--bob.md`, `isagi--all.md`
3. After writing, suggest committing the handoff file.

**Format (both modes):**

```markdown
# Handoff: <target>
**From:** <author>
**Date:** <today's date>

## <Short title of what changed>

### What changed
<1-3 bullets: what was done>

### What needs to happen
<Concrete action items>

### Breaking changes
<List any, or "None">
```

If the target file already exists (previous unread dispatch), **append** under a `---` separator. Don't overwrite unread dispatches.

### `handoff check`

Read pending dispatches.

**Solo mode:**
1. List all `.md` files in `~/.claude/handoffs/`
2. Summarize each: from, date, action items, breaking changes

**Team mode:**
1. List all `.md` files in `<repo>/.claude/handoffs/`
2. Filter to dispatches targeting you (match your git username or "all")
3. Also show dispatches targeting your team if identifiable from context
4. Summarize each: from, date, action items, breaking changes
5. If no dispatches for you, say "No pending dispatches for you. N dispatches for others exist."

### `handoff done <file-or-target>`

Delete a dispatch after executing it. The simple lifecycle.

**Solo mode:**
1. Delete `~/.claude/handoffs/<target>.md`
2. Confirm what was deleted.

**Team mode:**
1. Delete `<repo>/.claude/handoffs/<file>.md`
2. Suggest committing the deletion: `git add -A .claude/handoffs/ && git commit -m "handoff: done — <summary>"`
3. Git history preserves the dispatch for future reference.

### `handoff verify`

Check if the current codebase matches the API contract declared in a handoff dispatch. Detects drift between what was promised and what exists.

1. **Find the contract** — read the most recent handoff dispatch (from `handoff check` or user-specified). Extract all API contract tables: endpoints, methods, routes, body shapes, response shapes, auth levels, status enums.
2. **Scan the codebase** — for each declared endpoint:
   - Find the route definition (search for path string in route files, controllers, handlers)
   - Find the handler/controller method
   - Extract the actual request body type/validation
   - Extract the actual response shape
   - Check auth decorator/middleware
3. **Diff declared vs actual:**

```
HANDOFF VERIFY: <dispatch title>

MATCH (N):
  ✓ POST /deposit/create-intent — route, body, response all match

DRIFT (N):
  ✗ GET /bets — declared response: BetDocument[], actual: { bets: BetDocument[], count: number }
    Location: <file:line>
    Handoff says: flat array
    Code says: wrapped object

MISSING (N):
  ? POST /deposit/testnet — declared in handoff but no route found in codebase

UNDOCUMENTED (N):
  + DELETE /bets/:id — exists in code but not in any handoff

VERDICT: <N match, N drift, N missing, N undocumented>
```

4. **If drift or missing found**, suggest either:
   - Fix the code to match the contract
   - Write a new handoff dispatch to update the contract
   - Ask which is correct (code or handoff)

---

## Lifecycle

```
write → check → execute the work → done (delete)
                                      │
                              git history = archive
```

That's it. No ack, no stale, no archive folder. Read it, do it, delete it.

---

## Team Setup

To initialize team mode in a repo:

```bash
mkdir -p .claude/handoffs
echo "*.md" > .claude/handoffs/.gitkeep  # keep dir in git
```

Add to `.gitignore` if you do NOT want handoffs committed (rare — defeats the purpose).

### Naming Convention

```
.claude/handoffs/
  isagi--frontend-team.md     # isagi → frontend team
  alice--bob.md               # alice → bob specifically
  bob--all.md                 # bob → everyone
```

Pattern: `<from>--<to>.md` (double dash separates author from target).

### Conflict Prevention

Multiple people writing handoffs simultaneously won't conflict because each author writes to their own file (`<author>--<target>.md`). Different authors = different files = no merge conflicts.

---

## Rules

- **Concise** — just the delta, not full context
- **Actionable** — always include concrete action items, not just "FYI"
- **Breaking changes prominent** — flag them clearly
- **Delete after done** — don't hoard dispatches. Git history is the archive.
- If appending to an existing dispatch, add a `---` separator with new date header
- In team mode, always suggest committing after write or done
