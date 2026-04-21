# NeuroRouter Pro

When AI fails, you keep going.

NeuroRouter Pro is a local context-engineering proxy that preserves the semantically correct vector to the result before requests hit the API. It sits between your tools (Claude Code, Codex, OpenClaw, Cursor, Aider) and the upstream API.

## Install

```bash
brew install obstalabs/tap/neurorouter-pro
```

Or download from [releases](https://github.com/obstalabs/neurorouter-pro-dist/releases/latest).

## Activate

```bash
neurorouter activate <your-license-key>
```

Get a key at [neurorouter.dev](https://neurorouter.dev/#pricing) or try free for 14 days.

Activation validates the new key before replacing the saved key. Offline licenses work until their signed expiry; if a key expires, NeuroRouter falls back to Free and prints a clear warning.

## Quick Start

```bash
# Claude mode
neurorouter proxy --protocol anthropic --target https://api.anthropic.com --api-key env:ANTHROPIC_API_KEY
ANTHROPIC_BASE_URL=http://localhost:4000 claude

# Codex / OpenAI mode
neurorouter proxy --target https://api.openai.com --api-key env:OPENAI_API_KEY
codex -c 'openai_base_url="http://127.0.0.1:4000"'

# Codex no longer recommends OPENAI_BASE_URL.
# Use openai_base_url in a Codex profile or a -c override like the example above.

# See what would be filtered without sending
neurorouter proxy --dry-run
```

## What Pro Does

**Keep your session alive:**
- Session multiplexing — Claude, Codex, and any OpenAI-compatible tool in one daemon
- Continuity repair — broken tool chains fixed before they become upstream 400s
- Binary content sanitization — terminal output, SSH results, and file reads with control characters cleaned before they reach the API, preventing permanent session corruption
- Proactive JSONL healing — orphaned entries repaired automatically
- Context rescue — work extracted before compaction or cooldown

**Keep context sharp:**
- Reasoning Continuity Score (RCS) — verifies decisions, constraints, and rejected approaches survive context shaping
- Vector-state preservation — carries the objective, chosen approach, constraints, rejections, current state, and blockers across turns
- Mutation receipts — shows what changed, which content classes were shaped, and how much signal remained
- Auto model routing — mechanical work on Haiku or GPT-4o-mini by default
- Context shaping — removes non-cacheable repetition while preserving load-bearing constraints
- Rate limit prediction — warns before lockout, suggests cheaper models
- Per-project cost attribution — track spend by repo/branch

**Protect your data:**
- Reversible secret redaction — credentials replaced outbound, restored inbound
- Sensitive path protection — configurable deny/allow lists
- Prompt injection detection — warns on suspicious content in tool results
- Privacy scrub — opt-in content sanitization

**Stay informed:**
- `neurorouter doctor` — preflight diagnostics
- `neurorouter summary` — daily report
- `neurorouter status` — live session dashboard
- `neurorouter export` — session summary as markdown
- `neurorouter suggest` — workflow pattern suggestions
- `neurorouter rescue` — extract work before session loss

## Troubleshooting

| Issue | Fix |
|-------|-----|
| License not recognized | `neurorouter activate <key>` then `neurorouter version` |
| License near expiry | Activate the renewed key before the date shown in the startup banner |
| License expired | NeuroRouter falls back to Free; run `neurorouter activate <new-key>` after renewal |
| Service deployment | Do not put `--license-key` in systemd `ExecStart`; use `EnvironmentFile=/etc/neurorouter/env` with mode `0600` and `NEUROROUTER_LICENSE=...` |
| Port conflict | `neurorouter proxy --listen 127.0.0.1:9120` |
| Protocol detection | `neurorouter proxy --protocol anthropic` |
| Session issues | `neurorouter doctor` then `neurorouter stats` |

### Secure service deployment

For long-running services, prefer the saved activation file or a protected environment file. Command-line flags can be visible in process lists and service metadata.

```ini
# /etc/systemd/system/neurorouter.service
[Service]
EnvironmentFile=/etc/neurorouter/env
ExecStart=/usr/local/bin/neurorouter proxy --listen 127.0.0.1:4000
```

```bash
sudo install -d -m 0750 /etc/neurorouter
sudo install -m 0600 /dev/null /etc/neurorouter/env
sudoedit /etc/neurorouter/env
```

`/etc/neurorouter/env`:

```bash
NEUROROUTER_LICENSE=ol_...
ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...
```

## Security

Your API keys never leave your machine. NeuroRouter runs locally, uses environment variables or client passthrough auth, and never stores, transmits, or logs credentials. Keys exist only in process memory for the duration of the upstream request. Nothing is written to disk, state.db, or audit logs. [Verify it yourself](https://github.com/obstalabs/neurorouter-pro-dist/blob/main/docs/trust-architecture.md) — or run `lsof -i -P | grep neurorouter` and confirm the only connections are to your configured upstream.

This is a structural difference from cloud LLM proxies. A [2026 study](https://arxiv.org/abs/2604.08407) found 26 LLM proxy services collecting user credentials. The [LiteLLM supply-chain breach](https://gambit.security/blog-post/a-single-operator-two-ai-platforms-nine-government-agencies-the-full-technical-report) (March 2026) compromised thousands of organizations including a $10B startup. NeuroRouter eliminates this class of risk entirely — there is no server to breach and no database to leak.

## Works with ContextSpectre

NeuroRouter Pro and [ContextSpectre](https://github.com/ppiankov/contextspectre) are complementary — two stages of the same anti-waste pipeline.

| Layer | Tool | What it does |
|---|---|---|
| **Real-time** | NeuroRouter Pro | Shapes requests before they hit the API — protects vector anchors, strips stale reads, collapses failed retries, snapshots, and progress noise. Prevents 400 errors. Redacts secrets. |
| **Post-hoc** | ContextSpectre | Analyzes session files after requests complete — finds cross-turn tangents, measures noise ratios, repairs chain integrity, exports decisions. |

In a 214-request session, NeuroRouter Pro shaped 42.9% of payload in real time while preserving live vector anchors. ContextSpectre found an additional 151.7K tokens of cross-turn waste that only becomes visible with full conversation history. Maximum coverage requires both.

```bash
brew install ppiankov/tap/contextspectre
contextspectre stats
```

## Free vs Pro

The free community edition is at [obstalabs/neurorouter](https://github.com/obstalabs/neurorouter). [Full comparison](https://github.com/obstalabs/neurorouter#free-vs-pro).

## Links

- [Product site](https://neurorouter.dev) · [Pricing](https://neurorouter.dev/#pricing) · [Free edition](https://github.com/obstalabs/neurorouter) · [Why we built this](https://obstalabs.dev/blog/ai-breaks)

## What is this repo?

This repo contains only release binaries and checksums. Source code is private. GitHub shows `Source code` assets on release pages — those contain only this README.

## License

Proprietary. [Obsta Labs LLC](https://obstalabs.dev).
