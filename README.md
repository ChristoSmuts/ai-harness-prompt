# AI Harness Prompts — How to Use This

> **What is this?** A set of copy-paste prompts that teach your AI coding assistant how to work on *your* project — like giving a new teammate a briefing pack on day one.

You do **not** need to understand code deeply to use these. Copy a prompt, paste it into your AI tool, and answer the questions it asks.

---

## Quick start (3 steps)

1. **Open your project** in Cursor, Copilot, Claude Code, Continue, or another AI coding tool.
2. **Pick the right prompt** from this folder (use the chart below if unsure).
3. **Copy the entire prompt** → paste into chat → answer the questions.

You do not need to edit the prompts yourself. The AI will ask what it needs.

---

## Which prompt do I need?

```
START HERE
    |
    v
Is this a brand-new setup (no harness yet)?
    |
   YES --> [Harness Prompt](harness-prompt.md)
    |
   NO
    |
    v
Did the AI make the SAME mistake twice?
    |
   YES --> Log it in failure-ledger.json --> [Harness Quick Update Prompt](harness-quick-update-prompt.md)
    |
   NO
    |
    v
Did you upgrade the framework, add auth, or change tools?
    |
   YES --> [Harness Full Update Prompt](harness-full-update-prompt.md)
    |
   NO
    |
    v
Using a free/local AI (Ollama, LM Studio) that forgets context?
    |
   YES --> [Harness Context Engineering Prompt](harness-context-engineering-prompt.md)
    |
   NO --> [Harness Full Update Prompt](harness-full-update-prompt.md) (quarterly refresh is a good habit)
```

| Situation | Use this prompt |
|-----------|-----------------|
| New project or first-time setup | [Harness Prompt](harness-prompt.md) |
| AI keeps repeating the same mistake | [Harness Quick Update Prompt](harness-quick-update-prompt.md) |
| Big project changes or every few months | [Harness Full Update Prompt](harness-full-update-prompt.md) |
| Local/smaller AI forgets between sessions | [Harness Context Engineering Prompt](harness-context-engineering-prompt.md) |
| Not sure | [Harness Prompt](harness-prompt.md) |

---

## What you'll get in your project

After running **[Harness Prompt](harness-prompt.md)**, your repo gets new instruction files for the AI. Examples:

| File or folder | What it does (plain English) |
|----------------|------------------------------|
| `AGENTS.md` | Short cheat sheet the AI reads at the start of every session |
| `.cursor/rules/` (if using Cursor) | Do's and don'ts for your codebase |
| `.agents/skills/` | How-to guides loaded only when doing specific tasks |
| `docs/harness/HARNESS.md` | Index of all harness files and which prompt to run next |
| `docs/harness/failure-ledger.json` | A simple log of AI mistakes — so the harness can improve |
| `docs/harness/HARNESS_CHANGELOG.md` | Record of harness changes over time |

**You will NOT get:**
- New features for your app (unless you ask for that separately afterward)
- Shell scripts or automation files in your project
- Changes to your product code from the harness prompts alone

---

## The 4 prompts explained

### 1. [Harness Prompt](harness-prompt.md) — "Set up my project"

| | |
|---|---|
| **When** | Once per project, or when starting fresh |
| **Time** | About 10–30 minutes (mostly the AI working; you approve a plan) |
| **You do** | Answer which AI tool you use (Cursor, Copilot, Ollama, etc.) and your model tier (frontier vs lightweight/local) |
| **AI does** | Reads your project, shows a plan with a token budget, writes harness files after you approve |

---

### 2. [Harness Quick Update Prompt](harness-quick-update-prompt.md) — "Fix a repeating mistake"

| | |
|---|---|
| **When** | The AI made the **same** mistake twice (wrong import, wrong button, missed accessibility, etc.) |
| **You do** | Add an entry to `docs/harness/failure-ledger.json` (template is created during bootstrap) |
| **AI does** | Reads the log, adds one small, targeted fix to the harness |

> **Tip:** You can ask the AI to add the failure-ledger entry for you. Describe what went wrong and say: "Log this in failure-ledger.json."

---

### 3. [Harness Full Update Prompt](harness-full-update-prompt.md) — "Refresh everything"

| | |
|---|---|
| **When** | Framework upgrade, new major dependency, new team member, or every few months |
| **AI does** | Re-checks your project against the harness, updates outdated rules, re-audits skills, refreshes security guidance |

---

### 4. [Harness Context Engineering Prompt](harness-context-engineering-prompt.md) — "Help my AI remember"

| | |
|---|---|
| **When** | Optional; especially helpful for free or local models (Ollama, LM Studio, Continue) |
| **AI does** | Sets up memory layers, session handoff files, and optional tools (Graphify, OpenViking, Headroom) |

---

## Context engineering and suggested tools

**Context engineering** is how you organize what your AI sees — rules, docs, memory, and tool output — so it loads the right information at the right time without overflowing the context window. That matters most for **local or smaller models** (Ollama, LM Studio, Continue) that forget between sessions and have tighter token limits.

Use **[Harness Context Engineering Prompt](harness-context-engineering-prompt.md)** to design a tiered context layer for your project (L0 cheat sheet → L1 task context → L2 detail on demand). The prompt writes docs under `docs/context/` and updates your harness — it does **not** install tools for you.

For step-by-step local installation (Docker and non-Docker), see **[Context Tool Install](context-tool-install.md)**.

### Suggested tools (optional)

These run **locally** on your machine. Pick one, several, or none — the harness works without them.

| Tool | What it does | Best when |
|------|----------------|-----------|
| **[Graphify](https://github.com/safishamsi/graphify)** | Builds a queryable knowledge graph from your codebase, docs, PDFs, and more. Code is parsed on-device; you query the graph instead of grepping everything. | Large repos, lots of docs, or you want persistent “how does this connect?” navigation without re-reading files every session. |
| **[OpenViking](https://github.com/volcengine/OpenViking)** | Context database with a filesystem-style layout (`viking://resources/`, `viking://user/memories/`, `viking://agent/skills/`). Supports L0/L1/L2 tiered loading and semantic search. | You want unified memory + resources + skills, hierarchical browsing, and session memory that evolves over time. |
| **[Headroom](https://github.com/chopratejas/headroom)** | Compresses bulky context (tool outputs, logs, RAG chunks, file dumps) before it reaches the LLM — often 60–95% token savings, reversible. | Tool-heavy workflows, long logs, or local models where every token counts. |

### How they fit together

```
Your project files + harness docs (AGENTS.md, skills, rules)
              |
              v
    +---------+---------+---------+
    | Graphify | OpenViking | Headroom |
    | (graph)  | (context DB)| (compress)|
    +---------+---------+---------+
              |
              v
         Your AI agent (Cursor, Copilot, Claude Code, Ollama, etc.)
```

- **Graphify** — “What relates to what?” (structure and relationships).
- **OpenViking** — “Where is memory stored and how do I load it in tiers?” (organized context store).
- **Headroom** — “This output is huge — shrink it before the model sees it.” (compression layer).

You can use Graphify for navigation, OpenViking for memory, and Headroom on large tool outputs in the same project. Start with one tool that matches your biggest pain (repeated grepping → Graphify; scattered memory → OpenViking; token overflow → Headroom).

---

## Glossary

| Term | Simple meaning |
|------|----------------|
| **Harness** | The collection of instructions and rules that guide the AI on your project |
| **Skill** | A how-to file the AI loads only when doing a specific kind of task |
| **Rule** | An always-on constraint (e.g. "never commit secrets") |
| **failure-ledger** | A log of mistakes — so the harness can learn from them |
| **Model tier** | **Frontier** = powerful cloud models (GPT-4, Claude). **Lightweight** = smaller or free local models |
| **Token budget** | How much text the AI loads every session — smaller models need a thinner harness |
| **MCP** | A way for AI tools to connect to external services — the harness includes safety rules for these |
| **Context engineering** | Designing how context is stored, loaded in tiers, and refreshed so agents stay within token limits |
| **L0 / L1 / L2** | Context tiers: L0 = minimal cheat sheet, L1 = task-relevant summary, L2 = full detail on demand |

---

## Tips for juniors

- **Read the plan before saying yes.** The AI shows what it will create. You can ask it to change anything.
- **You don't need to understand every file.** Start with `AGENTS.md` if you're curious.
- **Log mistakes.** If the AI fails twice the same way, add it to `failure-ledger.json` and run [Harness Quick Update Prompt](harness-quick-update-prompt.md).
- **Ask a senior to review** harness files before merging to main if you're on a team.
- **One task at a time** works best with smaller/local models.

---

## FAQ

**Do I need Cursor?**  
No. These prompts work with Copilot, Claude Code, Codex, Continue, Ollama, Aider, and most tools that read `AGENTS.md`.

**Will this write code for my app?**  
No. Only AI instruction files (rules, skills, docs).

**Can I use this on an existing project?**  
Yes. The AI inspects what's already there and adapts.

**What if I picked the wrong AI tool at setup?**  
Run [Harness Full Update Prompt](harness-full-update-prompt.md) and correct it.

**What is failure-ledger.json?**  
A JSON file in your repo where you (or the AI) note repeated mistakes. [Harness Quick Update Prompt](harness-quick-update-prompt.md) reads it to add fixes.

**Do I need to run Context Engineering?**  
Optional. Recommended if you use a local or free model that forgets context between chats.

---

## Related files in this folder

- [Harness Prompt](harness-prompt.md) — initial setup
- [Harness Quick Update Prompt](harness-quick-update-prompt.md) — fix repeating mistakes
- [Harness Full Update Prompt](harness-full-update-prompt.md) — full refresh
- [Harness Context Engineering Prompt](harness-context-engineering-prompt.md) — memory and context layer
- [Context Tool Install](context-tool-install.md) — local install guide for Graphify, OpenViking, and Headroom
