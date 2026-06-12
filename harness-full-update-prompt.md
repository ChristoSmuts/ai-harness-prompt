---
tags:
  - harness
  - ai-agents
related:
  - "README.md"
  - "harness-quick-update-prompt.md"
  - "harness-context-engineering-prompt.md"
---

# Harness Full Update

You audit and refresh the **full AI harness** in this repository.

Do **not** build application features. Produce an **Update Plan**, get approval, then apply changes.

> User guide: [Harness Prompts README](README.md)

---

## CORE PURPOSE

Keep the harness aligned with how the project has grown — new dependencies, framework upgrades, new AI tools, outdated rules, and security surface changes.

For targeted fixes from repeated mistakes only, tell the user to run **Harness Quick Update** instead.

**Suggested cadence:** quarterly, after a major framework migration, when adding a new major dependency, when onboarding a new AI tool, or when adding team members.

---

## PHASE 0 — INTAKE

1. Read `docs/harness/HARNESS.md` for model tier, target agents, and last updated date
2. If `HARNESS.md` is missing, tell the user to run **Harness Prompt** first
3. Ask the user which update mode applies (single select):

| Option | When |
|--------|------|
| `standard` | Regular refresh — stack may have evolved gradually |
| `post-major-migration` | Framework or major version upgrade (e.g. Next 14 → 15) |
| `add-new-agent` | Team added a new AI tool (e.g. now using Cursor + Claude Code) |

4. Re-confirm **model tier** (`frontier` | `lightweight-local` | `lightweight-cloud`) if not recorded or if the user changed models

---

## PHASE 1 — DISCOVERY (DELTA SINCE LAST HARNESS)

Re-scan the repository. Compare against harness claims.

**Manifests & config:**
- `package.json`, lockfiles, workspace configs, framework configs
- New or removed dependencies vs what rules/skills reference

**Harness files:**
- `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`, rules, skills
- `docs/harness/SKILLS_PROVENANCE.md`, `failure-ledger.json`

**Checks:**
- Flag stack items in harness that no longer exist in the project
- Flag new stack items with no harness coverage
- `AGENTS.md` line count — warn if **>80 lines** (target ~60)
- Compare `.agents/skills/` (canonical) vs `.cursor/skills/`, `.claude/skills/` mirrors — content diff, not scripts
- Read `failure-ledger.json` for unresolved patterns — may fold into this update

**MCP & context:**
- `.cursor/mcp.json`, project MCP config
- `docs/context/CONTEXT_MANIFEST.md` if Context Engineering was run

Present findings as a **Drift Report** before the full Update Plan.

---

## PHASE 2 — SKILLS.SH FRESHNESS & SECURITY RE-AUDIT

For each skill listed in `SKILLS_PROVENANCE.md` with `usage: installed`:

1. Re-fetch from skills.sh if source URL/id is recorded
2. Re-run the **Phase 3.5 security audit** from **Harness Prompt** (prompt injection, execution risk, exfiltration, scope creep, provenance)
3. Check for newer curated alternatives at `GET https://skills.sh/api/v1/skills/curated`
4. Update `SKILLS_PROVENANCE.md` with re-audit date and outcome

For skills with `usage: merged-into-project-skill`, re-audit merged content only.

**Do not** install standalone `security-review` from skills.sh — regenerate the **project-derived** `security-review` skill (see Phase 4).

Rate limit: 60 req/min unauthenticated — batch sensibly.

---

## PHASE 3 — RULES & SKILLS HYGIENE

1. Dedupe guidance repeated across multiple rule files
2. Demote long rule sections to skills where appropriate (on-demand loading)
3. Prune stale rules referencing removed stack (old ORM, deprecated auth lib, etc.)
4. Apply **"fail twice before adding"** — review `failure-ledger.json` for patterns that justify new rules

### Context budget (mandatory in Update Plan)

Recompute the **always-on token budget** using the same format as **Harness Prompt Phase 2**:

| Artifact | Agent(s) | Lines (est.) | Chars (est.) | Est. tokens | Load |
|----------|----------|--------------|--------------|-------------|------|
| ... | | | | | |

| Metric | Value |
|--------|-------|
| **Total always-on** | **~X tokens** |
| **Tier** | [from HARNESS.md] |
| **Ceiling** | lightweight-local: 3,000 \| lightweight-cloud: 4,000 \| frontier: 8,000 |
| **Status** | within budget / over — propose trims |

**Estimation:** `est_tokens = round(char_count / 4)` or `round(word_count * 1.3)`

**Count:** `AGENTS.md`, `CLAUDE.md`, copilot-instructions, `alwaysApply` rules only.  
**Exclude:** on-demand skills, glob-scoped rules, `reference.md` files.

If over ceiling for lightweight tiers, include **Proposed trims** table and do not apply until user approves trims.

---

## PHASE 4 — MCP & SECURITY SURFACE

1. Re-scan MCP config — update `mcp-hygiene` rule allowlist with servers found
2. Flag unknown MCP servers as security gaps in the Update Plan
3. **Regenerate project-derived `security-review/SKILL.md`** if any of these changed since last harness update:
   - Auth library or session pattern
   - `.env.example` / public env var prefixes
   - API routes or server actions structure
   - Client/server component boundaries
4. Re-audit any skills.sh content merged into the security skill
5. Scan all harness files for accidental secrets (must be none — placeholders only)

**Project-derived security skill policy:**
- Always write `.agents/skills/security-review/SKILL.md` from **this repo's** real files
- Cite auth paths, env vars, validation library, client-security examples
- skills.sh security content: merge-only after audit passes; never replace project specifics

---

## PHASE 5 — UPDATE PLAN & APPLY

Present structured **Update Plan** grouped by agent. Include:
- Drift Report summary
- Files to create / update / remove
- Context budget table
- Skills audit outcomes
- Security skill regeneration (yes/no + reason)

Wait for user approval unless they said "apply without asking."

**On apply:**
1. Update harness files for all selected agents
2. Write skills to `.agents/skills/` first, then mirror to agent-native dirs
3. Append `docs/harness/HARNESS_CHANGELOG.md`
4. Update `docs/harness/HARNESS.md` — `last_updated` date, model tier, agent list
5. Resolve applicable `failure-ledger.json` entries if fixes address them

Do not commit unless the user asks.

---

## PHASE 6 — HAND OFF

1. **Summary table** — file | change | reason
2. **Context budget** — final actual vs previous estimate (char count on written always-on files)
3. **Gaps** — what the user must fill manually
4. **Recommendations** — run **Harness Context Engineering** if context pain noted; run **Harness Quick Update** for any remaining single-occurrence failures
5. **Next refresh** — suggest when to run Full Update again

---

## CONSTRAINTS

- No application feature code
- No shell scripts or hook files generated
- Minimal scope per file — respect model tier line limits from `HARNESS.md`
- Merge with existing harness; note overwrites
- Only write files for agents listed in `HARNESS.md` (or re-confirmed in Phase 0)

---

## START

Begin with **Phase 0**. Read `docs/harness/HARNESS.md`. Run **Phase 1** discovery and present the Drift Report before the full Update Plan.

---

## RELATED PROMPTS

- [Harness Prompts README](README.md) — which prompt to use when
- [Harness Quick Update Prompt](harness-quick-update-prompt.md) — failure-ledger fixes only
- [Harness Prompt](harness-prompt.md) — initial bootstrap
- [Harness Context Engineering Prompt](harness-context-engineering-prompt.md) — memory and context layer
