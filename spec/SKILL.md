---
name: spec
description: Manage project specifications with BINDING and NON-BINDING sections. Use when user says "spec", wants to add/modify spec rules, or needs to check code against spec. Use proactively during plan review to verify proposed changes comply with BINDING spec sections (run `spec check`). Use proactively after implementing changes that establish new invariants to suggest binding them (run `spec bind`).
argument-hint: [bind <rule>|note <observation>|check|diff]
---

# Spec — Treaties of the Great Convention

Manage `spec/SPEC.md` with formal governance over what's ratified (BINDING) vs proposed (NON-BINDING).

**Spec directory:** `<project-root>/spec/`

---

## Commands

### `spec bind <rule>`

Propose a BINDING spec addition or change. These are ratified treaties — they govern implementation.

1. Read current `spec/SPEC.md`
2. Draft the proposed BINDING entry with clear, testable language
3. Show the diff to the user
4. **Ask for explicit approval** before writing — BINDING changes require ratification
5. If approved, add under the appropriate `[BINDING]` section
6. Append a one-line "why" comment: `<!-- why: <reason> -->`

**BINDING rules must be:**
- Testable (you can verify code follows them)
- Specific (no ambiguous language)
- Scoped (say exactly what they cover)

### `spec note <observation>`

Add a NON-BINDING note freely. These are proposals, observations, and considerations.

1. Read current `spec/SPEC.md`
2. Add the note under the appropriate `[NON-BINDING]` section
3. No approval needed — these are proposals, not law
4. Append a one-line "why" comment

### `spec check`

Verify current implementation against BINDING specs.

1. Read `spec/SPEC.md` and extract all `[BINDING]` rules
2. For each rule, check the codebase for compliance
3. Report:
   - **Compliant:** rules the code follows
   - **Violation:** rules the code breaks (with file:line references)
   - **Untestable:** rules that can't be verified from code alone
4. Suggest fixes for violations

### `spec diff`

Show what changed in the spec since last commit.

1. Run `git diff spec/SPEC.md` and `git log --oneline -5 -- spec/SPEC.md`
2. Summarize changes: new BINDING rules, modified rules, new NON-BINDING notes
3. Flag any BINDING changes that weren't explicitly approved

---

## Spec Format

```markdown
# Spec: <Project Name>

## <Domain Section>

### [BINDING] <Rule title>
<Rule description — clear, testable, specific>
<!-- why: <reason for this rule> -->

### [NON-BINDING] <Note title>
<Observation, proposal, or consideration>
<!-- why: <context> -->
```

## Rules

- **BINDING = law.** Code must follow it. Changes require approval.
- **NON-BINDING = proposal.** Informational. Can be promoted to BINDING later.
- Every spec entry gets a "why" — no rules without reasoning
- Prefer specific over comprehensive — one clear rule beats three vague ones
- If a BINDING rule is wrong, change it (with approval) — don't just ignore it
