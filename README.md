# TagOpen

**Self-hostable, channel-native AI teammate for Slack.**  
LLM-agnostic open source alternative to Claude Tag.

> One agent per channel, shared by the whole team. Persistent memory. Skill auto-creation. Ambient monitoring. No vendor lock-in.

---

## What makes this different

Most Slack AI bots are personal assistants — one context per user, isolated DMs. TagOpen flips this:

- **Channel-scoped identity** — one agent shared by everyone in `#engineering`. All users see the same context, pick up mid-thread.
- **Multi-user attribution** — every message is tagged `[@alice]` so the agent knows who said what and can follow up with the right person.
- **Agent-curated memory** — after each conversation, the agent decides what to persist to `MEMORY.md`. No noisy append-only logs.
- **Skill auto-creation** — after complex multi-step tasks, the agent writes a `SKILL.md` capturing what it learned. Institutional knowledge accumulates automatically.
- **Ambient heartbeat** — configurable proactive monitoring: the agent surfaces stale threads, approaching deadlines, and unresolved questions without being tagged.
- **File-based config** — each channel is a directory of Markdown files. Version-controllable, no UI required.
- **MCP-native tools** — plug in any MCP server per channel. Admins control exactly what each channel's agent can access.

---

## Quickstart

```bash
# 1. Clone and install
git clone https://github.com/Anil-matcha/tagopen
cd tagopen
pip install -e ".[dev]"

# 2. Configure
cp .env.example .env
# Fill in SLACK_BOT_TOKEN, SLACK_APP_TOKEN, ANTHROPIC_API_KEY

# 3. Set up your first channel config
mkdir -p data/channels/YOUR_CHANNEL_ID
cp channels/example/CHANNEL.md data/channels/YOUR_CHANNEL_ID/CHANNEL.md
# Edit CHANNEL.md to describe your channel's purpose

# 4. Run
tagopen
```

Then `@tagopen` in your channel.

---

## Channel configuration

Each channel gets a directory under `data/channels/<channel_id>/`:

```
data/channels/C01234ABC/
  CHANNEL.md      ← identity, purpose, tone
  MEMORY.md       ← agent-maintained facts (do not edit manually)
  tools.toml      ← which MCP servers are enabled
  skills/         ← auto-created skill playbooks
    deploy.md
    oncall.md
```

### CHANNEL.md example
```markdown
# Engineering Channel

You are the engineering team's AI teammate.
Be concise, technical, and ask before triggering deploys.

## Team context
- Stack: Python, React, PostgreSQL, AWS
- We do not deploy on Fridays
```

### tools.toml example
```toml
[[mcp_server]]
name = "github"
url = "mcp://localhost:3001"
allowed_tools = ["list_prs", "get_file", "create_comment"]
```

---

## Architecture

```
Slack (Socket Mode)
       ↓
  Bolt Gateway
       ↓
  Channel Router  ← (workspace_id, channel_id) → AgentSession
       ↓
  Context Assembler  ← CHANNEL.md + MEMORY.md + skills + recent msgs
       ↓
  Agent Loop (ReAct + tool-use via LiteLLM)
       ├── Tool Registry (built-ins + MCP)
       ├── Streaming reply → Slack thread
       ├── Memory curation turn (Letta inner loop)
       └── Skill auto-creation (Hermes pattern)
       ↓
  SQLite + FTS5  ← per-channel message store
       ↓
  Ambient Engine  ← heartbeat cron, proactive posts
```

---

## Supported LLMs

Uses [LiteLLM](https://github.com/BerriAI/litellm) — swap provider via `LLM_MODEL` in `.env`:

| Provider | Model string |
|---|---|
| Anthropic (default) | `claude-sonnet-4-6` |
| OpenAI | `gpt-4o` |
| Gemini | `gemini/gemini-2.0-flash` |
| Local (Ollama) | `ollama/llama3` |

---

## Development

```bash
# Run tests
pytest

# Lint
ruff check .

# Type check
mypy tagopen/
```

---

## Roadmap

- [x] Phase 1 — Channel-native reactive teammate
- [ ] Phase 2 — Mem0 semantic recall + skill curator
- [ ] Phase 3 — Ambient heartbeat + agent-managed crons
- [ ] Phase 4 — Admin web UI + token governance
- [ ] Phase 5 — Discord + Teams adapters

See [PLAN.md](PLAN.md) for full architecture and design decisions.

---

## License

MIT
