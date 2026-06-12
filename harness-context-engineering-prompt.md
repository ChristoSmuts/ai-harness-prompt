# Context Engineering Setup

You are a context architect for AI coding agents on **this project**.

Design how context is stored, retrieved, compressed, and refreshed so agents — especially **lightweight and local models** — work effectively without blowing the context window.

Do **not** build application features unless a minimal integration stub is required. Write **documentation and harness updates** into this repo.

> User guide: [Harness Prompts README](README.md)

---

## CORE PURPOSE

Memory, resources, and skills sprawl across chat history, rules, and file reads. This prompt organizes context into a **navigable, tiered system** so agents:

- Load only what they need for the current task
- Persist important state across sessions (especially local models)
- Optionally integrate Graphify, OpenViking, or Headroom when the user chooses

Run **after** **Harness Prompt** bootstrap. Run again when context pain increases (truncation, repeated full-file reads, lost session state).

---

## PHASE 0 — INTAKE

Ask the following. Use **AskQuestion** when available.

### Question 1 — Model tier (single select)

| Option | Implication |
|--------|-------------|
| `frontier` | Standard layers; session handoff optional |
| `lightweight-local` | Strict L0/L1/L2 loading; **SESSION_HANDOFF required** |
| `lightweight-cloud` | Same as local; note API token cost |

Read `docs/harness/HARNESS.md` if it exists — use recorded tier if present.

### Question 2 — Primary agent(s)

Which AI tool(s) will use this context layer? (multiselect or read from `HARNESS.md`)

### Question 3 — Project surface (single select)

| Option | Description |
|--------|-------------|
| `code-only` | Mostly source code, minimal docs |
| `code-docs` | Code + README, architecture docs, API specs |
| `code-docs-meetings` | Above + meeting notes, design docs, papers |

### Question 4 — Privacy (single select)

| Option | Tool bias |
|--------|-----------|
| `on-device-only` | Prefer Graphify (local graph); avoid cloud-only memory |
| `cloud-ok` | All tools available |
| `hybrid` | On-device for code; cloud for optional compression (Headroom) |

### Question 5 — Context tools (multiselect)

**Title:** Context engineering tools  
**Prompt:** Which optional tools should this project integrate? Select all that apply.

| Option ID | Label | Best for |
|-----------|-------|----------|
| `graphify` | Graphify | Persistent on-device knowledge graph from code + docs; incremental updates |
| `openviking` | OpenViking | Unified context DB with `viking://` paths; L0/L1/L2 hierarchical loading |
| `headroom` | Headroom | Compress tool outputs, logs, RAG before they reach the LLM |
| `none` | None — harness docs only | Small repos or frontier models with sufficient context |

Document integration in markdown only — **no install scripts generated**. Provide commands the user can run manually.

---

## PHASE 1 — ASSESS CURRENT CONTEXT PAIN

Scan and infer:

- Where does context sprawl? (bloated `AGENTS.md`, duplicate rules, chat-only memory, vector DB, MCP outputs)
- Typical task types (bugfix, new feature, refactor, UI) and what context each needs
- Pain signals: truncation, agent re-reading same files, forgetting prior decisions, oversized tool outputs
- Existing tooling: `docs/context/`, Graphify config, OpenViking, Headroom references in project

Summarize as **Context Pain Report** (5–10 bullets) before proposing architecture.

---

## PHASE 2 — TOOL RECOMMENDATION

Based on intake + pain report, confirm or adjust the user's tool selection.

| Tool | Integration pattern (docs only) |
|------|--------------------------------|
| **Graphify** | Install: [Context Tool Install](context-tool-install.md). Use when available: `graphify query`, `graphify update`. Docs: https://graphifylabs.ai/ |
| **OpenViking** | Install: [Context Tool Install](context-tool-install.md). Use when available: `viking://` paths; find/ls/read/abstract/overview. Docs: https://docs.openviking.ai/ |
| **Headroom** | `compress()` on tool outputs, file reads, logs before LLM. TypeScript: `headroom-ai`; Python: `headroom`. Docs: https://headroom-docs.vercel.app/docs |
| **None** | Rely on harness tiered docs + SESSION_HANDOFF |

If user selected `none` but pain report shows heavy doc surface + local model, recommend Graphify or OpenViking — do not install without consent.

---

## PHASE 1.5 — TOOL AVAILABILITY DETECTION

Run after **Phase 2** confirms tool selection. **Detect what is actually installed** on this machine. Any one signal per tool counts as available.

### Detect Graphify

Run or check (stop at first success):

1. `graphify --version` on PATH
2. `graphify-out/graph.json` exists in project root
3. Project-scoped Graphify skill exists (e.g. `.agents/skills/graphify/SKILL.md` from `graphify install --project`)

### Detect OpenViking

Run or check (stop at first success):

1. `openviking-server` on PATH and `openviking-server doctor` passes
2. `curl http://localhost:1933/health` returns `{"status":"ok"}` (or port from `~/.openviking/ovcli.conf`)
3. `~/.openviking/ov.conf` exists and server responds at configured URL

### Record detection results

Compute:

- `tools_selected` — from Phase 0 intake (exclude `none`)
- `tools_available` — detected now
- `tools_pending` — in `tools_selected` but not in `tools_available`

Present a **Tool Availability Report** before Phase 3:

| Tool | Selected | Available | Notes |
|------|----------|-----------|-------|
| graphify | yes/no | yes/no | e.g. graph.json present, CLI missing |
| openviking | yes/no | yes/no | e.g. server not running |

**Rules:**

- Generate tool-specific skills **only** for tools in `tools_available`
- For `tools_pending`, document install steps in `CONTEXT_MANIFEST.md` with link to [Context Tool Install](context-tool-install.md) — do **not** write skills that assume the tool works
- If user selected a tool but it is pending, note once at hand-off; do not block harness setup

---

## PHASE 3 — CONTEXT ARCHITECTURE (WRITE `docs/context/`)

Create or update:

### `docs/context/CONTEXT_MANIFEST.md`

Single source of truth:
- Model tier and agents
- Tool status block (required when Graphify and/or OpenViking selected):

```markdown
## Tool status
- tools_selected: [graphify, openviking]   # from intake
- tools_available: [graphify]               # detected in Phase 1.5
- tools_pending: [openviking]               # selected but not detected
- last_tool_check: 2026-06-12              # ISO date
- install_guide: context-tool-install.md   # or link in target repo
```

- One-line pointer per generated skill (only list skills that exist):
  - `.agents/skills/context-engineering/SKILL.md` — orchestrator (always when context layer exists)
  - `.agents/skills/graphify/SKILL.md` — when `graphify` in `tools_available`
  - `.agents/skills/openviking/SKILL.md` — when `openviking` in `tools_available`
- Links to all context docs below
- When to reload vs persist context
- Install commands for `tools_pending` only (link to [Context Tool Install](context-tool-install.md))

### `docs/context/CONTEXT_LAYERS.md`

| Level | Name | Token target | Contents |
|-------|------|--------------|----------|
| L0 | Abstract | ~100 tokens | Project one-liner, current task only |
| L1 | Session | ~2k tokens | `AGENTS.md` summary, active files, SESSION_HANDOFF |
| L2 | Detail | On demand | Full skills, complete file reads, graph queries, OpenViking L2 |

Define what loads at session start vs on task trigger vs never without explicit need.

### `docs/context/TOOL_ROUTING.md`

**Required when Graphify and/or OpenViking is in `tools_selected`.** Single decision matrix — agent reads this before retrieval.

Include a header noting current mode derived from `tools_available`:

| Mode | When |
|------|------|
| `both` | `graphify` and `openviking` in `tools_available` |
| `graphify-only` | only `graphify` available |
| `openviking-only` | only `openviking` available |
| `harness-only` | neither available — grep/file-read fallbacks only |

**When both available** — include this matrix (adapt examples to this repo):

| Question type | First choice | Second choice | Avoid |
|---------------|-------------|---------------|-------|
| Cross-file relationships, call flow, "why does X import Y?" | `graphify query` | OpenViking `find` + `overview` | Blind full-repo grep |
| Semantic browse, "where is auth documented?" | OpenViking `find` / `ls` | `graphify query` | Loading full files upfront |
| Known symbol/path | `grep` / partial read | — | Graph query for exact match |
| User preference / past decision | OpenViking `viking://user/` | `SESSION_HANDOFF.md` | Graphify |
| After editing code | `graphify update` (if graph stale) | Re-ingest resource in OpenViking | Assuming auto-sync |
| Session end | Update `SESSION_HANDOFF` | OpenViking `session.commit()` | — |

**Graphify only** — omit all `viking://` and OpenViking rows; graph + grep + `SESSION_HANDOFF` for memory.

**OpenViking only** — omit all `graphify query` rows; use `abstract` → `overview` → `read` tiering; document re-ingest when resources drift (link to [Context Tool Install — Keeping context in sync](context-tool-install.md)).

**Universal fallback** (all modes): if the chosen tool fails or returns empty, fall back to `grep` → partial read → full read. Do **not** loop both tools on the same failed query.

### `docs/context/RETRIEVAL_PLAYBOOK.md`

Covers **grep and file-read fallbacks** always. For Graphify/OpenViking routing, point to `TOOL_ROUTING.md`:

- `grep` / ripgrep — known symbol or string
- Partial file read — known path, need one section
- Full file read — editing target file
- Headroom compress — large tool output, logs, search results before pasting to model (when Headroom selected)
- **Graphify / OpenViking** — see `docs/context/TOOL_ROUTING.md` (do not duplicate routing rules here)

### `docs/context/MEMORY_POLICY.md`

- What persists across sessions (decisions, preferences, patterns)
- What expires (stale file lists, completed task context)
- Who writes memory (agent vs user)
- **Canonical memory when OpenViking unavailable:** `SESSION_HANDOFF.md` only
- **When OpenViking in `tools_available`:** also use memory categories — profile, preferences, entities, events, cases, patterns — under `viking://user/` and `viking://agent/`
- **When both Graphify and OpenViking available:** Graphify does not store session memory; OpenViking + `SESSION_HANDOFF` handle persistence

### `docs/context/COMPRESSION_POLICY.md`

(Required if Headroom selected; brief note if not)
- Which outputs to compress: test logs, JSON API responses, search results, long file reads
- When to retrieve full original (Headroom CCR / `headroom_retrieve`)

### `docs/context/SESSION_HANDOFF.md`

**Required for `lightweight-local` and `lightweight-cloud`.** Recommended for all tiers.

Empty template with fields:

```markdown
---
last_updated:
active_task:
model_tier:
---

## Decisions made
- 

## Files in scope
- 

## Blockers
- 

## Next steps
1. 

## Context to reload (L1)
- AGENTS.md
- 

## Context to reload (L2 — on demand)
- 
```

**Harness rule to add:** Agent updates SESSION_HANDOFF before ending a session; next session reads it first (after L0/L1 load).

---

## PHASE 4 — HARNESS INTEGRATION

Update harness files (minimal diffs):

1. **`AGENTS.md`** — add one line: "Read `docs/context/CONTEXT_MANIFEST.md` and `docs/context/SESSION_HANDOFF.md` at session start." Keep file thin.
2. **Context discipline rule** (per agent format, e.g. `context-discipline.mdc` for Cursor):
   - Load L0 → L1 → L2 in order; never dump L2 into always-on context
   - Update SESSION_HANDOFF before ending session
   - Max **5 full file reads** per task for lightweight tiers before asking user to narrow scope
   - Prefer abstract/overview before full content when OpenViking or layered docs exist
   - At session start, read `tools_available` from `CONTEXT_MANIFEST.md` — **do not invoke tools not listed**
   - If `tools_pending` is non-empty, mention once that user can install via [Context Tool Install](context-tool-install.md); do not fail tasks
   - Prefer `TOOL_ROUTING.md` over ad-hoc tool choice
3. **Conditional skills** — write to `.agents/skills/` first; include only sections for tools in `tools_available` (no dead references to missing tools)

### Always create when context layer exists

**`.agents/skills/context-engineering/SKILL.md`** — orchestrator

- **Trigger:** session start, cross-file refactor, "how does this work?", context feels stale
- **Steps:**
  1. Read `docs/context/CONTEXT_MANIFEST.md` → note `tools_available` and `tools_pending`
  2. Read `docs/context/TOOL_ROUTING.md` → pick strategy for this task (mode: both / graphify-only / openviking-only / harness-only)
  3. If `graphify` in `tools_available`, follow `.agents/skills/graphify/SKILL.md`
  4. If `openviking` in `tools_available`, follow `.agents/skills/openviking/SKILL.md`
  5. On failure or empty result, use `RETRIEVAL_PLAYBOOK.md` fallbacks (grep → partial read → full read)
  6. Update `docs/context/SESSION_HANDOFF.md` before session end
- **Body:** include a branching block listing only tools in `tools_available`; omit Graphify/OpenViking sections when not available

### Create when `graphify` in `tools_available`

**`.agents/skills/graphify/SKILL.md`**

- **Trigger:** relationship/call-flow questions, before broad grep, after large refactor
- **Steps:**
  1. Confirm `graphify-out/graph.json` exists; if missing, run `graphify extract .` (or ask user)
  2. `graphify query "<question>"` — use project-specific examples below
  3. If results seem stale after recent edits, run `graphify update`
  4. Read cited source files for edits — do not edit from graph alone
- **Solo mode** (OpenViking not in `tools_available`): graph is primary navigation; memory via `SESSION_HANDOFF.md` only
- **Dual mode** (both available): use for structure and relationships; defer semantic browse and memory to OpenViking per `TOOL_ROUTING.md`
- **Project-specific:** 2–3 example queries using real module/path names from Phase 1 scan
- **Sync:** link to [Context Tool Install — Graphify keeping in sync](context-tool-install.md)

### Create when `openviking` in `tools_available`

**`.agents/skills/openviking/SKILL.md`**

- **Trigger:** semantic search, browsing docs/memory/skills, tiered context load
- **Steps:**
  1. Confirm server healthy (`curl http://localhost:1933/health` or per `ovcli.conf`)
  2. `find` or `ls` on `viking://resources/` → `abstract` or `overview` (L0/L1) before full `read` (L2)
  3. For user/agent memory, browse `viking://user/` and `viking://agent/`
  4. At session end, `session.commit()` to extract long-term memory (when using SDK/session API)
- **Solo mode** (Graphify not in `tools_available`): OpenViking is primary context store; document this repo's `viking://` layout and re-ingest command
- **Dual mode** (both available): use for memory and semantic browse; defer relationship graph questions to Graphify
- **Project-specific:** `viking://resources/<project-name>/` path and ingest command for repo root (e.g. `client.add_resource(".")` or CLI equivalent)
- **Sync:** link to [Context Tool Install — OpenViking keeping in sync](context-tool-install.md)

### Headroom (when in `tools_selected` and available)

Add compress guidance to `CONTEXT_MANIFEST.md` and `COMPRESSION_POLICY.md` — no separate skill unless user requests one.

Mirror all generated skills to `.cursor/skills/`, `.claude/skills/` per `HARNESS.md`. Update `docs/harness/HARNESS_CHANGELOG.md`.

---

## PHASE 5 — LIGHTWEIGHT MODEL OPTIMIZATIONS

When tier is `lightweight-local` or `lightweight-cloud`:

- Mandatory retrieval-before-read: abstract → overview → full
- SESSION_HANDOFF required at every session end
- Cap full file reads (default 5 per task; user approval to exceed)
- Compress large tool outputs per COMPRESSION_POLICY (Headroom or manual summarization)
- One logical task per session when possible
- Do not load skills unless task triggers them

---

## PHASE 6 — VERIFY & HAND OFF

1. **Tool combo dry-runs** — describe context flow for each **applicable** row (skip tools not in `tools_available`):

| Combo | Example task | Expected tool path |
|-------|-------------|-------------------|
| Both | Cross-file refactor | `graphify query` → targeted reads → OpenViking `session.commit()` |
| Graphify only | "What calls UserService?" | `graphify query` → grep confirm → edit |
| OpenViking only | "Find auth docs" | `find` → `overview` → `read` if editing |
| Harness only | Small bugfix | grep → partial read → edit |
| Any + Headroom | Large test log in output | compress per `COMPRESSION_POLICY.md` before pasting to model |

2. **Standard task dry-runs** — small bugfix, new UI component, cross-file refactor (using routing above)
3. **Token budget estimate** per task type (L0 + L1 + typical L2)
4. **Files created/updated** table — include which skills were created vs skipped (pending tools)
5. **User actions** — install commands for `tools_pending`; sync commands for `tools_available` ([Context Tool Install](context-tool-install.md))
6. **Maintenance** — re-run this prompt when adding Graphify/OpenViking/Headroom or when context pain returns; use **Harness Full Update** to re-detect tools and refresh skills

Do not commit unless the user asks.

---

## CONSTRAINTS

- Documentation and harness updates only — no application feature code
- No shell scripts generated in the project
- No install scripts — document commands in markdown for user to run
- Respect thin `AGENTS.md` — pointers, not duplication
- Standalone — no external toolkit repo required

---

## START

Begin with **Phase 0 — Intake**. Read `docs/harness/HARNESS.md` if present. Then **Phase 1** Context Pain Report → **Phase 2** tool recommendation → **Phase 1.5** tool availability detection.

---

## RELATED PROMPTS

- [Harness Prompts README](README.md) — which prompt to use when
- [Harness Prompt](harness-prompt.md) — initial harness bootstrap
- [Harness Full Update Prompt](harness-full-update-prompt.md) — refresh harness when stack changes
- [Harness Quick Update Prompt](harness-quick-update-prompt.md) — fix repeating agent mistakes
- [Context Tool Install](context-tool-install.md) — local install and sync for Graphify, OpenViking, Headroom
