# NeuroRouter Pro

Context compiler for AI coding agents.

NeuroRouter Pro runs locally between Claude Code, Codex, OpenAI-compatible tools, and the model API. It compiles messy agent history into minimal, model-ready context while preserving the active reasoning vector and reporting whether a long session is still trustworthy to continue before requests hit the API.

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
neurorouter proxy --listen 127.0.0.1:9120 --client-profile codex --api-key env:OPENAI_API_KEY
codex -p nr

# Put this profile in your Codex config:
# [model_providers.neurorouter]
# name = "NeuroRouter"
# base_url = "http://127.0.0.1:9120"
# openai_base_url = "http://127.0.0.1:9120"
# wire_api = "responses"
#
# [profiles.nr]
# model_provider = "neurorouter"
# model = "gpt-5.4"
#
# Resume and model override:
# codex -p nr resume SESSION_ID
# codex -p nr -m gpt-5.5 resume SESSION_ID
#
# OPENAI_BASE_URL alone is not reliable for current Codex routing.
# Use --auth-source client_passthrough only when the client forwards Authorization.

# See what would be filtered without sending
neurorouter proxy --dry-run
```

## What Pro Does

**Keep your session alive:**
- Session multiplexing — Claude, Codex, and any OpenAI-compatible tool in one daemon
- Continuity repair — broken tool chains repaired locally when safe, or blocked before they become upstream 400s
- Binary content sanitization — terminal output, SSH results, and file reads with control characters cleaned before they reach the API, reducing the risk of permanent session corruption
- Proactive JSONL healing — orphaned entries repaired when the structure is provably safe
- Context rescue — work extracted before compaction or cooldown

**Compile context:**
- Context compiler — source transcript to semantic field to target model context
- Reasoning Continuity Score (RCS) — verifies decisions, constraints, and rejected approaches survive context shaping
- Vector Lock — carries the objective, chosen approach, constraints, rejections, current state, and blockers across turns and restarts
- Session Integrity — downgrades false-green sessions when objective freshness, workspace identity, recovery, loop, progress, or tool-chain signals fail
- Workspace Identity Lock — keeps the active repo, path, remote, and release target explicit after compression or restart
- Mutation receipts — shows what changed, which content classes were shaped, and how much signal remained
- Auto model routing — mechanical work on Haiku or GPT-4o-mini by default
- Context shaping — removes non-cacheable repetition while preserving load-bearing constraints and current work
- Rate limit prediction — warns before lockout, suggests cheaper models
- Per-project cost attribution — track spend by repo/branch

Vector Lock is not chat memory, RAG, or learning. It is the compact local constraint set that keeps the model on the correct path when compaction, shaping, or proxy restarts would otherwise erase load-bearing context. NeuroRouter compiles that constraint set into the next request without keeping the whole transcript verbatim.

RCS is useful, but it is not the whole health model. Session Integrity can invalidate a green RCS when the active objective is stale, the workspace lock conflicts, recovery disabled major filters, or loop/progress signals show the agent is stuck.

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
- `neurorouter integrity` — summarize healthy, degraded, and critical session integrity from support-safe session evidence
- `neurorouter rescue` — extract work before session loss

## When Pro Helps

NeuroRouter Pro is most useful when AI coding sessions get long, expensive, or
fragile. It keeps useful context alive, removes stale transcript drag, protects
detected secrets, repairs safe tool-chain continuity breaks before provider 400
errors, and warns when a session is no longer making trustworthy progress.

## What It Is Not

NeuroRouter Pro is not a replacement for the model, a hosted gateway, memory,
RAG, or an autonomous agent brain. It is a local compiler and safety layer for
the context you already send.

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

Your provider API keys are forwarded only to the upstream provider you configure. NeuroRouter runs locally, uses environment variables or client passthrough auth, and is designed not to store provider keys on disk, in state.db, or in audit logs. Keys exist in process memory for the duration of the upstream request. [Verify it yourself](https://github.com/obstalabs/neurorouter-pro-dist/blob/main/docs/trust-architecture.md) — or run `lsof -i -P | grep neurorouter` and confirm the only outbound connections are to your configured upstream.

This is a structural difference from cloud LLM proxies. A [2026 study](https://arxiv.org/abs/2604.08407) found 26 LLM proxy services collecting user credentials. The [LiteLLM supply-chain breach](https://gambit.security/blog-post/a-single-operator-two-ai-platforms-nine-government-agencies-the-full-technical-report) (March 2026) compromised thousands of organizations including a $10B startup. A local proxy removes the hosted credential database from this path; it is not a promise to catch every encoded, chunked, or transformed secret.

## Works with ContextSpectre

NeuroRouter Pro and [ContextSpectre](https://github.com/ppiankov/contextspectre) are complementary — two stages of the same anti-waste pipeline.

| Layer | Tool | What it does |
|---|---|---|
| **Real-time** | NeuroRouter Pro | Compiles requests before they hit the API — protects vector anchors, strips stale reads, collapses failed retries, snapshots, and progress noise. Repairs safe tool-chain breaks or blocks unsafe ones locally. Redacts detected secrets. |
| **Post-hoc** | ContextSpectre | Analyzes session files after requests complete — finds cross-turn tangents, measures noise ratios, repairs chain integrity, exports decisions. |

In a 214-request session, NeuroRouter Pro shaped 42.9% of payload in real time while preserving live vector anchors. ContextSpectre found an additional 151.7K tokens of cross-turn waste that only becomes visible with full conversation history. Maximum coverage requires both.

```bash
brew install ppiankov/tap/contextspectre
contextspectre stats
```

## Free vs Pro

The free community edition is at [obstalabs/neurorouter](https://github.com/obstalabs/neurorouter). [Full comparison](https://github.com/obstalabs/neurorouter#free-vs-pro).

## Links

- [Product site](https://neurorouter.dev) · [Pricing](https://neurorouter.dev/#pricing) · [Free edition](https://github.com/obstalabs/neurorouter) · [Context compiler essay](https://obstalabs.dev/blog/context-compiler) · [Why we built this](https://obstalabs.dev/blog/ai-breaks)

## What is this repo?

This repo contains only release binaries and checksums. Source code is private. GitHub shows `Source code` assets on release pages — those contain only this README.

## License

Proprietary. [Obsta Labs LLC](https://obstalabs.dev).
