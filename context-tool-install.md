# Context Tool Install — Graphify, OpenViking, Headroom

> **What is this?** Step-by-step instructions to run optional context-engineering tools **locally** on your machine (not cloud-hosted). Use alongside **[Harness Context Engineering Prompt](harness-context-engineering-prompt.md)** after your harness is set up.

You do **not** need all three tools. Install only what you need. Each section works standalone.

---

## Before you start

| Requirement | Graphify | OpenViking | Headroom |
|-------------|:--------:|:----------:|:--------:|
| Python 3.10+ | Yes | Yes | Yes (or Docker-only) |
| Node.js 18+ | No | No (optional for Rust CLI) | Optional (TypeScript SDK) |
| Docker | Optional (MCP server) | Optional (full stack) | Optional (proxy/runtime) |
| LLM / embedding API or Ollama | Optional* | **Yes** | No (compresses existing text) |
| Disk space | ~500 MB–2 GB+ for graph output | ~1 GB+ for workspace + models | ~500 MB+ with ML extras |

\* Graphify parses **code locally** with no API key. Semantic extraction for docs, PDFs, and images needs an LLM (your IDE session, or Ollama / API keys for headless `graphify extract`).

**Recommended package managers**

- **[uv](https://docs.astral.sh/uv/)** — fastest way to install Python CLI tools (`uv tool install …`)
- **pipx** — isolated Python CLIs without uv
- **Docker Desktop** (or Docker Engine on Linux) — if you prefer containers over installing Python deps on the host

Run installs from your **project root** unless noted otherwise.

---

## Graphify

**Docs:** [github.com/safishamsi/graphify](https://github.com/safishamsi/graphify) · [graphifylabs.ai](https://graphifylabs.ai/)

Graphify turns your project folder into a **queryable knowledge graph** (code, docs, PDFs, images, video). Code is parsed on-device with tree-sitter. Query with `graphify query` instead of grepping the whole repo.

> **Important:** The official PyPI package is **`graphifyy`** (two y's). The CLI command is still `graphify`. Other `graphify*` packages on PyPI are unrelated.

### What you need

- **Python 3.10+**
- **uv** or **pipx** (strongly recommended over plain `pip` on Windows/macOS)
- **Optional:** API key or [Ollama](https://ollama.ai) for semantic extraction of non-code files
- **Optional:** `ffmpeg` if you install the `[video]` extra for local audio/video transcription
- **Disk:** Graph output lives in `graphify-out/` (typically hundreds of MB for medium repos)

### Non-Docker install (recommended)

**1. Install the CLI**

```bash
# Recommended — puts graphify on PATH automatically
uv tool install graphifyy

# Alternatives
pipx install graphifyy
pip install graphifyy   # may need ~/.local/bin on PATH (Linux) or Python Scripts (Windows)
```

**Windows (uv via winget):**

```powershell
winget install astral-sh.uv
uv tool install graphifyy
```

**2. Register the skill with your AI assistant**

From your project directory:

```bash
# User-wide (default)
graphify install

# Project-scoped (commits skill into the repo)
graphify install --project

# Cursor-specific hook
graphify cursor install --project
```

See the [Graphify README](https://github.com/safishamsi/graphify) for other platforms (`--platform codex`, `graphify claude install`, etc.).

**3. Build the graph**

Inside your AI tool, run the graphify skill (e.g. `/graphify .` in Claude Code). Or from the terminal:

```bash
cd /path/to/your/project
graphify extract .          # headless build
graphify query "How does auth work?"
graphify update             # incremental refresh after changes
```

**4. Optional extras**

```bash
uv tool install "graphifyy[video]"    # local transcription (faster-whisper)
uv tool install "graphifyy[pdf]"
uv tool install "graphifyy[office]"   # .docx, .xlsx
uv tool install "graphifyy[mcp]"        # MCP server mode
uv tool install "graphifyy[ollama]"   # fully local semantic extraction
uv tool install "graphifyy[all]"      # everything
```

**5. Keeping context in sync**

Graphify serves a **cached graph** (`graphify-out/graph.json`). Queries read the cache, not live files — refresh after the codebase changes.

| Method | When to use | Command |
|--------|-------------|---------|
| Incremental update | Day-to-day after edits | `graphify update` |
| Full rebuild | Major refactor or corrupted graph | `graphify extract .` |
| Git hooks | Automatic on commit / branch switch | `graphify hook install` |
| Live watch | Active development session | `graphify watch .` |

```bash
graphify update              # preferred — only changed files since last run
graphify hook install        # rebuild on git commit / branch switch
graphify watch .             # optional — continuous sync while editing
```

Your harness `TOOL_ROUTING.md` should tell the agent to run `graphify update` when query results look stale after recent edits.

### Docker install (MCP server mode)

Docker is mainly for running Graphify as an **HTTP MCP server** that serves an existing `graph.json`. You still build the graph on the host (or mount a pre-built graph).

**1. Clone and build the image**

```bash
git clone https://github.com/safishamsi/graphify.git
cd graphify
docker build -t graphify .
```

**2. Build or copy graph output to `graphify-out/`**

On the host (non-Docker):

```bash
uv tool install graphifyy
cd /path/to/your/project
graphify extract .
```

**3. Run the MCP server container**

```bash
docker run -p 8080:8080 \
  -v "$(pwd)/graphify-out:/data" \
  graphify /data/graph.json --transport http --host 0.0.0.0 --api-key "$SECRET"
```

Point your MCP client at `http://localhost:8080/mcp` with the API key. Bind to `0.0.0.0` only on trusted networks and always set `--api-key`.

### Verify Graphify

```bash
graphify --version
graphify query "What is the entry point?" --json
ls graphify-out/graph.json
```

---

## OpenViking

**Docs:** [docs.openviking.ai](https://docs.openviking.ai/) · [github.com/volcengine/OpenViking](https://github.com/volcengine/OpenViking)

OpenViking is a **context database** for agents: memories, resources, and skills under `viking://` paths, with L0/L1/L2 tiered loading and semantic search.

### What you need

- **Python 3.10+**
- **Embedding model** — for vector search (Ollama, OpenAI, or other [supported providers](https://docs.openviking.ai/en/guides/01-configuration))
- **VLM model** — for content understanding (API provider or local via Ollama)
- **Disk:** workspace data under `~/.openviking/` or a path you configure (1 GB+ typical)
- **RAM:** depends on models; local Ollama models often need 8 GB+ system RAM for small embeddings + a capable VLM

For a **fully local** stack, install [Ollama](https://ollama.ai) and use `openviking-server init` — the wizard can detect Ollama and pull suitable models.

### Non-Docker install (recommended)

**1. Install OpenViking**

```bash
# Recommended
uv tool install openviking --upgrade

# Alternative
pip install openviking --upgrade --force-reinstall
```

**2. Run the setup wizard**

```bash
openviking-server init
```

The wizard prompts for embedding and VLM providers. For local-only use, choose **Ollama** when offered. It can install Ollama and pull models if missing.

**3. Validate configuration**

```bash
openviking-server doctor
```

Fixes connectivity, config paths, and model access before you start the server.

**4. Start the server**

```bash
openviking-server
# Default: http://127.0.0.1:1933
```

**5. Configure the CLI client**

Create `~/.openviking/ovcli.conf`:

```json
{
  "url": "http://localhost:1933",
  "api_key": "your-root-api-key-from-ov.conf"
}
```

**6. Smoke test**

```bash
curl http://localhost:1933/health
# {"status": "ok"}

python -m openviking ls viking://
```

**Optional Rust CLI:**

```bash
npm i -g @openviking/cli
# or: cargo install --git https://github.com/volcengine/OpenViking ov_cli
```

Config file location: `~/.openviking/ov.conf` (created by `init`). See the [configuration guide](https://docs.openviking.ai/en/guides/01-configuration) for embedding/VLM provider options.

### Docker install (local container)

OpenViking publishes a image to GitHub Container Registry. All state lives under `/app/.openviking` inside the container — mount your host `~/.openviking` to persist it.

**1. Prepare config on the host**

```bash
# If you don't have ov.conf yet, install the CLI once and run init on the host:
uv tool install openviking --upgrade
openviking-server init
openviking-server doctor
```

Ensure `ov.conf` includes a `root_api_key` under `server` (required when the container binds `0.0.0.0`):

```json
{
  "server": {
    "root_api_key": "your-secret-root-key"
  }
}
```

**2. Run the container**

```bash
docker run -d \
  --name openviking \
  -p 1933:1933 \
  -v ~/.openviking:/app/.openviking \
  --restart unless-stopped \
  ghcr.io/volcengine/openviking:latest
```

**Without a host volume** (config only via env):

```bash
docker run -d \
  --name openviking \
  -p 1933:1933 \
  -e OPENVIKING_CONF_CONTENT="$(cat ~/.openviking/ov.conf)" \
  --restart unless-stopped \
  ghcr.io/volcengine/openviking:latest
```

**If the container starts before `ov.conf` exists:**

```bash
docker exec -it openviking openviking-server init
```

**3. Optional: docker compose**

Clone the repo and use the bundled compose file:

```bash
git clone https://github.com/volcengine/OpenViking.git
cd OpenViking
docker compose up -d
```

**4. Access**

- API: `http://localhost:1933`
- Web Studio UI: `http://localhost:1933/studio`
- Health: `curl http://localhost:1933/health`

**Note:** If you use **Ollama on the host** while OpenViking runs in Docker, point `api_base` in `ov.conf` at the host gateway (e.g. `http://host.docker.internal:11434` on Docker Desktop, or your LAN IP on Linux).

### Keeping context in sync

OpenViking has **two update paths** — resources (code/docs you ingest) and memory (extracted from agent sessions). They do not auto-sync with git.

**Resources** (`viking://resources/`) — re-ingest when source files change:

```bash
# Python SDK example — re-add project root after significant changes
python -c "
import openviking as ov
client = ov.SyncHTTPClient(url='http://localhost:1933', api_key='your-key')
client.initialize()
client.add_resource('.')   # or path to docs subtree
client.close()
"

# Or via API
curl -X POST http://localhost:1933/api/v1/resources \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"path": "/path/to/your/project"}'
```

Ingestion is **asynchronous** — check status via API before assuming search results are current. Consider re-ingesting after merge to main or on a schedule you document in `docs/context/TOOL_ROUTING.md`.

**Memory** (`viking://user/`, `viking://agent/`) — evolves from sessions, not from file edits:

```python
session = client.session()
session.add(role="user", content="...")
session.add(role="assistant", content="...")
session.commit()   # extracts long-term memory automatically
```

Use `session.commit()` at session end for preferences and task experience. Pair with harness `SESSION_HANDOFF.md` when Graphify handles code navigation separately.

| What changed | Action |
|--------------|--------|
| Source code / docs on disk | Re-ingest resource |
| Agent learned a preference | `session.commit()` |
| Both Graphify + OpenViking | `graphify update` for graph; re-ingest for OpenViking resources |

### Verify OpenViking

```bash
openviking-server doctor
curl http://localhost:1933/ready
python -m openviking ls viking://resources/
```

---

## Headroom

**Docs:** [headroom-docs.vercel.app](https://headroom-docs.vercel.app/docs) · [github.com/chopratejas/headroom](https://github.com/chopratejas/headroom)

Headroom **compresses context** (tool outputs, logs, RAG chunks, chat history) before it reaches the LLM — typically 60–95% token reduction, with reversible retrieval of originals.

### What you need

- **Python 3.10+** for the full compression pipeline (recommended install: `headroom-ai[all]`)
- **OR Docker** — no Python on the host required
- **Optional:** Node.js 18+ if you use the TypeScript SDK (SDK talks to the Python proxy over HTTP)
- **Optional:** PyTorch (~2 GB) if you use the `[ml]` extra (Kompress text compression); `[all]` includes it
- **No LLM API key** — Headroom only shrinks text you already have; your agent still needs its own model

### Non-Docker install (recommended)

**1. Install**

```bash
pip install "headroom-ai[all]"

# Minimal / pick extras
pip install "headroom-ai[proxy]"   # proxy + MCP tools
pip install "headroom-ai[mcp]"       # MCP tools only
pip install "headroom-ai[code]"      # tree-sitter code compression
```

**TypeScript (optional):**

```bash
npm install headroom-ai
```

**2. Run as a local proxy (zero code changes)**

```bash
headroom proxy --port 8787
```

Point your LLM client at the proxy:

```bash
# Examples
export ANTHROPIC_BASE_URL=http://localhost:8787
export OPENAI_BASE_URL=http://localhost:8787/v1
```

**3. Wrap a coding agent (one command)**

```bash
headroom wrap cursor              # prints Cursor config instructions
headroom wrap claude              # starts proxy + launches Claude Code
headroom wrap codex
headroom wrap aider
headroom wrap claude --memory     # persistent cross-agent memory
```

**4. MCP server (Cursor, Claude Desktop, etc.)**

```bash
headroom mcp install
```

Exposes `headroom_compress`, `headroom_retrieve`, and `headroom_stats` to your MCP client.

**5. Library use in Python**

```python
from headroom import compress

result = compress(messages, model="claude-sonnet-4-5-20250929")
# result.messages — compressed; result.tokens_saved — savings
```

**6. Persistent background service (optional)**

```bash
headroom install apply --preset persistent-service --providers auto
```

### Docker install (local container)

**Option A — Official install script (Docker-native wrapper)**

Installs a host wrapper that runs Headroom in Docker; wrapped tools (Claude, Codex, Cursor, etc.) stay on the host.

**Linux / macOS (Bash 4.3+):**

```bash
curl -fsSL https://raw.githubusercontent.com/chopratejas/headroom/main/scripts/install.sh | bash
```

**Windows (PowerShell):**

```powershell
irm https://raw.githubusercontent.com/chopratejas/headroom/main/scripts/install.ps1 | iex
```

**Option B — Pull the image directly**

```bash
docker pull ghcr.io/chopratejas/headroom:latest
```

**Option C — Persistent Docker runtime**

```bash
# After Docker-native install script, or with pip + Docker available:
headroom install apply --preset persistent-docker
```

**Run proxy in Docker (manual):**

```bash
docker run --rm -p 8787:8787 ghcr.io/chopratejas/headroom:latest \
  headroom proxy --port 8787 --host 0.0.0.0
```

Then set `ANTHROPIC_BASE_URL=http://localhost:8787` (or `OPENAI_BASE_URL=http://localhost:8787/v1`) for your agent.

### Verify Headroom

```bash
headroom --version
headroom proxy --port 8787 &
curl -s http://localhost:8787/health || echo "Check proxy docs if no /health endpoint"
```

Quick Python check:

```python
from headroom import compress
r = compress([{"role": "user", "content": "x " * 5000}], model="gpt-4o")
print(r.compression_ratio, r.tokens_saved)
```

---

## Using tools with this harness

1. Run **[Harness Prompt](harness-prompt.md)** first if you have no harness yet.
2. Run **[Harness Context Engineering Prompt](harness-context-engineering-prompt.md)** and select which tools to integrate.
3. Install tools from this guide.
4. The context-engineering prompt writes `docs/context/CONTEXT_MANIFEST.md` with project-specific paths, `viking://` layout, and compression policy — follow that doc after install.

**Typical workflow**

| Pain | Tool | First command |
|------|------|----------------|
| Re-grepping the same files | Graphify | `graphify extract .` then `graphify query "…"` |
| Memory and resources scattered | OpenViking | `openviking-server init` → `openviking-server` |
| Huge tool output / logs | Headroom | `headroom proxy --port 8787` or `headroom mcp install` |

---

## Troubleshooting

| Issue | Tool | Fix |
|-------|------|-----|
| `graphify: command not found` | Graphify | Use `uv tool install graphifyy` or `pipx install graphifyy`; avoid plain `pip` on Windows/macOS |
| Wrong package installed | Graphify | Uninstall `graphify`, install **`graphifyy`** (two y's) |
| OpenViking won't start in Docker | OpenViking | Set `server.root_api_key` in `ov.conf`; mount `~/.openviking:/app/.openviking` |
| Ollama unreachable from Docker | OpenViking | Use `host.docker.internal:11434` (Desktop) or host IP in `api_base` |
| `openviking-server doctor` fails | OpenViking | Re-run `openviking-server init`; check embedding/VLM API keys or Ollama models |
| Headroom proxy connection refused | Headroom | Ensure proxy is running; check port 8787; TypeScript SDK needs Python proxy |
| Large download on install | Headroom | Use `pip install headroom-ai` (core only) instead of `[all]` if you skip ML compression |

---

## Related files

- [README](README.md) — overview and which harness prompt to use
- [Harness Context Engineering Prompt](harness-context-engineering-prompt.md) — design your project's context layer
- [Graphify](https://github.com/safishamsi/graphify) · [OpenViking](https://docs.openviking.ai/) · [Headroom](https://headroom-docs.vercel.app/docs)
