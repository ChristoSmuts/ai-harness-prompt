---
tags:
  - harness
  - ai-agents
related:
  - "README.md"
  - "harness-full-update-prompt.md"
---

# Harness Quick Update

You improve the AI harness in **this repository** based on the failure ledger.

Do **not** build application features. Make **minimal harness changes only** — one small fix per repeated failure pattern.

> User guide: [Harness Prompts README](README.md)

---

## CORE PURPOSE

When an AI agent makes the **same mistake twice**, this prompt reads `docs/harness/failure-ledger.json`, identifies the pattern, and adds the smallest harness fix that prevents a third occurrence.

For a full harness refresh (stack changes, new dependencies, quarterly audit), tell the user to run **Harness Full Update** instead.

---

## PHASE 0 — CONFIRM SCOPE

State clearly:

> "This is a **Quick Update** — failure-ledger driven only. For a full stack refresh, use **Harness Full Update**."

If the user asked for both (e.g. "update harness and fix mistakes"), prioritize failure-ledger fixes first. Recommend Full Update separately if stack drift is also evident.

---

## PHASE 1 — READ FAILURE SIGNAL

Read these files (create missing ones only if the harness was never bootstrapped — in that case, tell the user to run **Harness Prompt** first):

- `docs/harness/failure-ledger.json`
- `docs/harness/HARNESS.md`
- `docs/harness/HARNESS_CHANGELOG.md`

**Stop and ask the user** if:
- `failure-ledger.json` does not exist → run **Harness Prompt** first
- Ledger is empty → ask user to log at least one failure (or describe one now so you can add it)
- No pattern has **≥2 entries** with the same `pattern` label → explain the "fail twice before fixing" rule; offer to log the current failure and wait for a second occurrence, or suggest **Harness Full Update** if the issue is broader than a single pattern

---

## PHASE 2 — PATTERN ANALYSIS

1. Group `failure-ledger.json` entries by `pattern` field
2. Consider only patterns with **≥2 open entries** (same `pattern`, `status: "open"`)
3. For each qualifying pattern, determine the **minimal fix type**:
   - Single bullet in an existing rule file
   - One new step in an existing skill
   - One-line pointer in `AGENTS.md`
   - New rule only if no existing file covers the domain

**Do not:**
- Add large new skills preemptively
- Rewrite unrelated harness files
- Fix patterns with only one logged occurrence (document as "watching" in hand-off instead)

---

## PHASE 3 — PLAN (BRIEF)

Show a compact table and wait for approval unless the user said "apply without asking":

| Pattern | Occurrences | Proposed fix | File | Est. lines added |
|---------|-------------|--------------|------|------------------|
| wrong-import-path | 3 | Add import boundary bullet | `.cursor/rules/typescript.mdc` | +2 |

If multiple patterns qualify, fix **at most 3 per run** — prioritize highest occurrence count.

---

## PHASE 4 — APPLY

1. Edit harness files — **canonical source is `.agents/skills/`** when skills exist; mirror changes to `.cursor/skills/`, `.claude/skills/`, etc. as documented in `HARNESS.md`
2. Append entry to `docs/harness/HARNESS_CHANGELOG.md`:
   ```markdown
   ## [YYYY-MM-DD] Quick Update — [pattern]
   - **Pattern:** [label]
   - **Fix:** [one sentence]
   - **Files:** [paths]
   ```
3. Update resolved entries in `failure-ledger.json`:
   - Set `status: "resolved"`
   - Set `fix_ref` to the changelog heading or file path

Do not commit unless the user asks.

---

## PHASE 5 — HAND OFF

1. **What changed** — file, pattern fixed, why this fix should prevent recurrence
2. **One test prompt** — a short task the user can run to verify the fix (e.g. "Add a form to the settings page" if the pattern was form-related)
3. **Watching** — patterns with only 1 entry so far
4. **Escalation** — if failures suggest stack drift (new dependency unused in harness, wrong framework version in rules), recommend **Harness Full Update**

---

## FAILURE-LEDGER SCHEMA

When adding entries (user request or during this session):

```json
{
  "entries": [
    {
      "id": "unique-id",
      "timestamp": "2026-06-12T10:00:00Z",
      "pattern": "short-kebab-label",
      "description": "What the agent did wrong",
      "task_context": "What the user asked for",
      "files_involved": ["src/components/example.tsx"],
      "status": "open",
      "fix_ref": null
    }
  ]
}
```

**Pattern naming:** use consistent labels across entries (e.g. `invented-component`, `wrong-toast-library`, `missed-form-label`).

---

## CONSTRAINTS

- Minimal diff — prefer editing existing files over creating new ones
- One minimal fix per pattern per run
- No application feature code
- No shell scripts
- Respect model tier constraints documented in `HARNESS.md` (keep always-on context thin)

---

## START

Begin with **Phase 0**. Read `docs/harness/failure-ledger.json`. If viable patterns exist, proceed to **Phase 2**.

---

## RELATED PROMPTS

- [Harness Prompts README](README.md) — which prompt to use when
- [Harness Full Update Prompt](harness-full-update-prompt.md) — full harness refresh
- [Harness Prompt](harness-prompt.md) — initial bootstrap if no harness exists
