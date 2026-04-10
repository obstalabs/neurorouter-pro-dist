# NeuroRouter Pro

When AI fails, you keep going.

NeuroRouter Pro is a local proxy that cleans, repairs, and controls AI requests before they hit the API. It sits between your tools (Claude Code, Codex, OpenClaw, Cursor, Aider) and the upstream API.

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

## Quick Start

```bash
# Claude mode
neurorouter proxy --protocol anthropic --target https://api.anthropic.com --api-key env:ANTHROPIC_API_KEY
ANTHROPIC_BASE_URL=http://localhost:4000 claude

# Codex / OpenAI mode
neurorouter proxy --target https://api.openai.com --api-key env:OPENAI_API_KEY
codex -c 'openai_base_url="http://127.0.0.1:4000"'

# See what would be filtered without sending
neurorouter proxy --dry-run
```

## What Pro Does

**Keep your session alive:**
- Session multiplexing — Claude, Codex, and any OpenAI-compatible tool in one daemon
- Continuity repair — broken tool chains fixed before they become upstream 400s
- Proactive JSONL healing — orphaned entries repaired automatically
- Context rescue — work extracted before compaction or cooldown

**Save money:**
- Auto model routing — mechanical work on Haiku or GPT-4o-mini by default
- Token filtering — compounds via snowball to ~$60 saved per 200-turn session
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
| Port conflict | `neurorouter proxy --listen 127.0.0.1:9120` |
| Protocol detection | `neurorouter proxy --protocol anthropic` |
| Session issues | `neurorouter doctor` then `neurorouter stats` |

## Free vs Pro

The free community edition is at [obstalabs/neurorouter](https://github.com/obstalabs/neurorouter). [Full comparison](https://github.com/obstalabs/neurorouter#free-vs-pro).

## Links

- [Product site](https://neurorouter.dev) · [Pricing](https://neurorouter.dev/#pricing) · [Free edition](https://github.com/obstalabs/neurorouter) · [Why we built this](https://obstalabs.dev/blog/ai-breaks)

## What is this repo?

This repo contains only release binaries and checksums. Source code is private. GitHub shows `Source code` assets on release pages — those contain only this README.

## License

Proprietary. [Obsta Labs LLC](https://obstalabs.dev).
