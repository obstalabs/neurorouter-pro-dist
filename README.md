# NeuroRouter Pro

Local context shaping and safety layer for Claude Code, Codex, and
OpenAI-compatible coding tools.

NeuroRouter Pro runs on your machine between your coding agent and the model
provider. It keeps long sessions usable, trims stale transcript drag, protects
detected secrets, and warns when a session is no longer healthy enough to keep
going.

## Quick Start

```bash
brew install obstalabs/tap/neurorouter-pro
nr activate <your-license-key>
nr launch claude
```

Use Codex the same way:

```bash
nr launch codex
```

`nr launch <client>` is the recommended OAuth path. It starts the local
NeuroRouter proxy, chooses a loopback port, prepares the client environment, and
writes customer-safe launcher logs automatically.

## Privacy Modes

Launcher logs are customer-safe by default. Detected secrets default to `warn`,
which alerts you before the request continues but does not rewrite the request
payload.

For strict privacy mode, turn on outbound redaction before launching:

```bash
nr config set protect_policy redact
nr config set output_secret_policy redact
nr launch claude
```

Manual proxy and API-key workflows keep their configured policy. Use strict mode
when you want detected secrets replaced before provider requests.

The release installs both `nr` and `neurorouter`; examples use `nr`.

## Install

```bash
brew install obstalabs/tap/neurorouter-pro
```

Or download from [releases](https://github.com/obstalabs/neurorouter-pro-dist/releases/latest).

## Activate

```bash
nr activate <your-license-key>
```

Get a key at [neurorouter.dev](https://neurorouter.dev/#pricing) or try free for
14 days.

Activation validates the new key before replacing the saved key. Offline
licenses work until their signed expiry; if a key expires, NeuroRouter falls
back to Free and prints a clear warning.

## What Pro Does

**Keep your session alive:**
- Session multiplexing for Claude, Codex, and OpenAI-compatible tools
- Tool-chain repair when local continuity breaks can be fixed safely
- Binary content sanitization for terminal output, SSH results, and file reads
- Proactive JSONL healing when the structure is provably safe
- Context rescue before compaction or cooldown

**Shape context:**
- Context shaping for long, noisy coding-agent sessions
- Session health checks for task continuity, workspace identity, recovery state,
  loops, progress, and tool-chain reliability
- Workspace identity guard for active repo, path, remote, and release target
- Progress receipts that show what changed and how much useful context remained
- Auto model routing for mechanical work on cheaper models by default
- Rate-limit prediction before lockout
- Per-project cost attribution by repo and branch

The goal is not to memorize your chats or replace the model. NeuroRouter keeps a
compact local task frame so compaction, restarts, and long tool sessions do not
erase the constraints that matter.

**Protect your data:**
- Reversible secret redaction, with credentials restored only on the local reply
  path
- Sensitive path protection with configurable deny and allow lists
- Prompt injection detection for suspicious tool output
- Optional privacy scrub for additional content sanitization

**Stay informed:**
- `nr doctor` - preflight diagnostics
- `nr summary` - daily report
- `nr status` - live session dashboard
- `nr export` - session summary as markdown
- `nr suggest` - workflow pattern suggestions
- `nr integrity` - support-safe session health summary
- `nr rescue` - extract work before session loss

## Advanced Proxy Mode

Use `nr launch <client>` for OAuth-backed Claude and Codex sessions. Manual proxy
mode is for API-key workflows, service deployments, and advanced operators who
need direct control over ports, targets, and client configuration.

Stable Claude API-key proxy:

```bash
neurorouter proxy --protocol anthropic --target https://api.anthropic.com --api-key env:ANTHROPIC_API_KEY
ANTHROPIC_BASE_URL=http://localhost:4000 claude
```

Stable Codex API-key proxy:

```bash
neurorouter proxy --listen 127.0.0.1:9120 --client-profile codex --api-key env:OPENAI_API_KEY
codex -p nr
```

Codex profile example:

```toml
[model_providers.neurorouter]
name = "NeuroRouter"
base_url = "http://127.0.0.1:9120"
openai_base_url = "http://127.0.0.1:9120"
wire_api = "responses"

[profiles.nr]
model_provider = "neurorouter"
model = "gpt-5.4"
```

Anthropic OAuth passthrough is experimental; volume/cooldown rate limits may
surface as 429s NR cannot recover from. Prefer `nr launch claude` for OAuth.

```bash
neurorouter proxy --protocol anthropic --target https://api.anthropic.com --auth-source client_passthrough
ANTHROPIC_BASE_URL=http://localhost:4000 claude
```

`OPENAI_BASE_URL` alone is not reliable for current Codex routing. Use a Codex
provider profile, or use `nr launch codex`.

To preview filtering without sending traffic upstream:

```bash
neurorouter proxy --dry-run
```

## When Pro Helps

NeuroRouter Pro is most useful when AI coding sessions get long, expensive, or
fragile. It keeps useful context alive, removes stale transcript drag, protects
detected secrets, repairs safe tool-chain continuity breaks before provider
errors, and warns when a session is no longer making trustworthy progress.

## What It Is Not

NeuroRouter Pro is not a replacement for the model, a hosted gateway, memory,
RAG, or an autonomous agent brain. It is a local compiler and safety layer for
the context you already send.

## Troubleshooting

| Issue | Fix |
|-------|-----|
| License not recognized | `nr activate <key>` then `nr version` |
| License near expiry | Activate the renewed key before the date shown in the startup banner |
| License expired | NeuroRouter falls back to Free; run `nr activate <new-key>` after renewal |
| Claude or Codex OAuth setup | Use `nr launch claude` or `nr launch codex` |
| Service deployment | Do not put `--license-key` in systemd `ExecStart`; use `EnvironmentFile=/etc/neurorouter/env` with mode `0600` and `NEUROROUTER_LICENSE=...` |
| Port conflict | `neurorouter proxy --listen 127.0.0.1:9120` |
| Protocol detection | `neurorouter proxy --protocol anthropic` |
| Session issues | `nr doctor` then `nr status` |

### Secure service deployment

For long-running services, prefer the saved activation file or a protected
environment file. Command-line flags can be visible in process lists and service
metadata.

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

Your provider API keys are forwarded only to the upstream provider you configure.
NeuroRouter runs locally, uses environment variables or client passthrough auth,
and is designed not to store provider keys on disk, in its local state file, or
in audit logs. Keys exist in process memory for the duration of the upstream
request.

This is a structural difference from cloud LLM proxies. A
[2026 study](https://arxiv.org/abs/2604.08407) found 26 LLM proxy services
collecting user credentials. The
[LiteLLM supply-chain breach](https://gambit.security/blog-post/a-single-operator-two-ai-platforms-nine-government-agencies-the-full-technical-report)
(March 2026) compromised thousands of organizations including a $10B startup. A
local proxy removes the hosted credential database from this path; it is not a
promise to catch every encoded, chunked, or transformed secret.

## Works with ContextSpectre

NeuroRouter Pro and
[ContextSpectre](https://github.com/ppiankov/contextspectre) are complementary
stages of the same anti-waste pipeline.

| Layer | Tool | What it does |
|---|---|---|
| **Real-time** | NeuroRouter Pro | Shapes requests before they hit the API, preserves task-critical context, trims stale reads, collapses failed retries, repairs safe tool-chain breaks, and redacts detected secrets. |
| **Post-hoc** | ContextSpectre | Analyzes session files after requests complete, finds cross-turn tangents, measures noise ratios, repairs chain structure, and exports decisions. |

```bash
brew install ppiankov/tap/contextspectre
contextspectre stats
```

## Free vs Pro

The free community edition is at
[obstalabs/neurorouter](https://github.com/obstalabs/neurorouter). See the
[full comparison](https://github.com/obstalabs/neurorouter#free-vs-pro).

## Links

- [Product site](https://neurorouter.dev)
- [Pricing](https://neurorouter.dev/#pricing)
- [Free edition](https://github.com/obstalabs/neurorouter)
- [Context compiler essay](https://obstalabs.dev/blog/context-compiler)
- [Why we built this](https://obstalabs.dev/blog/ai-breaks)

## What is this repo?

This repo contains only release binaries and checksums. Source code is private.
GitHub shows `Source code` assets on release pages; those contain only this
README.

## License

Proprietary. [Obsta Labs LLC](https://obstalabs.dev).
