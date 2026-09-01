# NeuroRouter Pro Customer Changelog

This changelog is the source for customer-facing GitHub release notes in
`obstalabs/neurorouter-pro-dist`.

Keep this file focused on customer outcomes, operator action, compatibility,
trust, and safety. Do not copy private developer changelog entries here
verbatim, and do not mention internal function names, filter names, detector
names, thresholds, ranking logic, or precedence rules.

## [Unreleased]

## [0.38.10] - 2026-09-02

### Added
- **The Helm chart can now be installed by reference.** The chart is published alongside the container image, so it can be installed and upgraded directly instead of downloading a file from a release page first. The published chart is checked against the release asset before the release completes.
- **The container image and the Helm chart are documented.** The repository now describes pulling the image by tag or by digest, installing the chart, supplying credentials as Kubernetes Secrets, the single-instance requirement, and how to verify the files you downloaded.

### Fixed
- **The container image is listed with the distribution it belongs to**, so it can be found from the same place as the binaries and the chart.
- **Release listings no longer show internal build identifiers.**

## [0.38.9] - 2026-09-01

Version 0.38.8 delivered the signed binaries, Helm chart, and checksums. This
release adds the container image that accompanies them.

### Fixed
- **The gateway container image is now published alongside the release.** The first publication of a new image location could not complete, so the image was missing from an otherwise complete release. Publishing now succeeds on a first release, and a genuine registry or authentication error still stops the release rather than being mistaken for an empty registry.

## [0.38.8] - 2026-09-01

Version 0.38.7 was prepared but never published. If you are upgrading from
0.37.2, this release contains everything described in the sections below.

### Changed
- **The distribution repository now carries its own license.** Binaries, container images, and Helm charts published here are covered by the NeuroRouter Pro commercial license, included as `LICENSE` alongside the downloads. The separately published community edition remains under its own open-source license in its own repository; that license does not extend to these artifacts.
- **The repository documentation now describes what a downloaded package contains** and states the trust boundary the proxy operates under, so both are available before you install rather than only inside the archive.

## [0.38.7] - 2026-09-01

Version 0.38.6 was prepared but never published. If you are upgrading from
0.37.2, this release contains everything described in the sections below.

### Fixed
- **Signed releases now publish with a certificate you can verify.** The step that assembles a completed release required the signing certificate in a form the signer does not produce, so the release stopped before publishing rather than emit anything unverified. The certificate is now decoded to its standard form, and the same check runs both when the release is assembled and again before anything is published, so the two cannot disagree. A failure at this point leaves nothing partially written behind.

### Security
- **The signature verifier bundled with the release process is updated.** The pinned verifier moves to the vendor's authenticated security release, which addresses an advisory affecting the older verification path. Both the vendor's checksum file and the verifier binary must match their expected values before the release process will use it.

## [0.38.6] - 2026-09-01

Versions 0.38.0 through 0.38.5 were prepared but never published; 0.38.6 is the
first release that delivers the Kubernetes gateway work described below. If you
are upgrading from 0.37.2, this release contains everything in both sections.

### Fixed
- **Concurrent subagent activity no longer loses a record on Windows.** When many subagents started at once, a transient file-permission error from the operating system could cause one of them to be dropped from the recorded set while the others were kept. Windows self-update and subagent tracking now use an operating-system-level lock, so contention, a genuine permission failure, and a stopped process are distinguished from one another instead of being treated alike. If a lock left behind by an older version is found, NeuroRouter stops and tells you the exact file to remove rather than deleting it automatically — an old lock carries nothing that proves its owner has exited, and reclaiming it while a process is still running could lose exactly the records this change protects.

## [0.38.0] - 2026-08-28

### Added
- **Run NeuroRouter inside your own Kubernetes cluster.** A hardened container image and a Helm chart are now published with each release, so the router can sit next to your workloads instead of on a developer machine. The image runs as a non-root user on a minimal base with a read-only filesystem and no extra privileges, and the chart applies restricted Pod Security settings, a default-deny network policy, and provider credentials supplied only as existing Kubernetes Secrets — never written into chart values. Health and metrics are served on a separate port from model traffic, so probes and dashboards never touch the endpoint your agents use.
- **The gateway runs as exactly one instance, and the chart will not let you change that.** Token custody depends on a single process owning session state, so the replica count is not a setting you can override — an attempt to scale it is rejected when the chart is validated, rather than silently accepted and producing two processes that disagree. The chart README explains the one situation this does not cover: if a node partition is uncertain, do not force-delete the pod until the old process is confirmed stopped.
- **Every release artifact can be checked before you trust it.** Images are built reproducibly, so an independent rebuild of the same release produces an identical digest; archives, the chart, and package manifests are published together with checksums and signatures, and a release is only completed once all of them agree.

### Fixed
- **Shutdown no longer cuts off work in progress.** Model traffic, metrics, and long-lived streaming connections now drain together under one deadline, so a restart or upgrade finishes what it started instead of dropping connections mid-response.
- **Startup is tolerant of a slow or briefly unavailable work-order backend**, retrying rather than failing outright, while still refusing to attribute work to a project it cannot confirm.

### Security
- **Redacted values can no longer leak through a stream boundary.** Where a protected value was split across streaming chunks, or carried over from a previous run, the connection now ends with a clear error containing no content, rather than passing through partially-restored text. The same protection now applies to response headers, model-discovery responses, and error bodies.

## [0.37.2] - 2026-08-24

### Fixed
- **`nr tool-shape` returns your content exactly as it was.** When this command passed content through, long values such as identifiers, hashes and URLs could be shortened on the way out — an identifier could arrive with its ending replaced by an ellipsis. The command reported success, so a shortened value could be used without anyone noticing it had changed. Content now leaves the command byte-for-byte identical, and if it cannot be written in full the command reports an error instead of returning a shortened result. This affected only what this command wrote out; content sent to a model provider was never altered by it, and the protection that keeps sensitive values out of ordinary command output and messages is unchanged.

## [0.37.1] - 2026-08-23

### Fixed
- **Windows: saving important state is far more reliable when another program has the file open.** On Windows, replacing a file that another program is reading can be refused by the operating system, where the same operation always succeeds on macOS and Linux. NeuroRouter now uses the Windows-native replace operation everywhere it saves state — approvals, licence state, logs, crash reports, and updates — with a short automatic retry for the brief conflicts a backup tool or security scanner typically causes, and an automated check that keeps every location using it. Worth knowing the limit: Windows decides file sharing when a program opens a file, and that decision cannot be overridden afterwards. If another program holds the file open in a mode that forbids replacement, the save still fails and reports a clear error rather than appearing to succeed. Updating the running program itself is handled separately, because Windows does not permit replacing an executable while it is running.
- **A recorded approval can no longer be lost.** When approvals were recorded at the same moment from more than one place, the last write could silently discard the earlier one — on Windows it surfaced as an error, on macOS and Linux it was silent. Approvals are now recorded one at a time across the whole machine, so every decision is kept. If a decision cannot be recorded, the command says so and exits with an error rather than reporting success.
- **Upgrading is safer if it is interrupted.** The updater now prepares the new binary completely before it touches the running one, and both recovery paths use the same protected replacement. An interrupted or failed update leaves the existing install in place with a clear error, instead of a partially replaced one. The upgraded binary is also verified to remain executable.
- **Crash reports no longer overwrite each other.** Two crashes in the same second previously produced one report, discarding the first — usually the one that explained the problem. Every crash report is now kept.
- **Log rotation cannot silently discard a log file.** Rotated logs are given non-colliding names, and a rotation that fails now reports the failure instead of continuing as if it had worked.
- **Error messages appear once.** Some failures were printed twice. Each error is now shown a single time.

### Note
- This release focuses on Windows reliability for the file operations NeuroRouter depends on. Behaviour on macOS and Linux is unchanged by design; no configuration changes are required on any platform, and no settings or commands were added or altered.


## [0.37.0] - 2026-08-21

### Added
- **You can now tell NeuroRouter where your repositories live.** NeuroRouter records which project each session belongs to, so its governance records — including the state it preserves when a long session is compacted — land in the right place. It only trusts paths inside the folders you declare as your working roots, which is what stops an unrelated repository on your machine from naming your session. The default is a `dev` folder in your home directory. If your code lives somewhere else, set `workspace_roots` in your configuration file, or the `NEUROROUTER_WORKSPACE_ROOTS` environment variable for a single session. Windows users should expect to set one of these during rollout, since the default reflects a common Unix layout. See the Workspace Roots section of the launch guide.

### Fixed
- **Sessions are no longer attributed to an unrelated repository.** NeuroRouter could name a session after any code repository it found referenced on your machine — including one belonging to a package manager rather than to your work. Attribution now only considers paths inside your declared working folders, so an unrelated repository can no longer claim your session.
- **Session records are never written to the wrong project.** When NeuroRouter could not determine which project a session belonged to, it previously fell back to a fixed project name and wrote there anyway. It now declines to write and records that it declined. Nothing is guessed, and no project receives another project's history.
- **The same protection covers the state preserved before a long session is compacted.** If the project cannot be determined at that moment, the save is skipped and recorded rather than filed under the wrong project — because a record stored in the wrong place is both lost to its owner and noise to everyone else.

### Note
- If NeuroRouter cannot determine your project, it says so at startup and tells you exactly which settings resolve it. Session syncing stays off until you set one; nothing is written in the meantime.

## [0.36.10] - 2026-08-12

### Fixed
- **Launching Codex with an API key no longer breaks provider-run tools without telling you.** Provider-run tools such as web search are only available on account sign-in launches, not API-key launches. Previously a session that used one would be refused with an unclear error and retry until it gave up. The launch now tells you up front that these tools are unavailable in this mode and which sign-in method does support them, and any refusal explains itself. If you want provider-run tools, launch without setting an API key.
- **Endpoint guidance now matches the provider you are actually using.** Pointing at a provider that speaks the OpenAI API format — but is not OpenAI — could produce sign-in advice meant for OpenAI, including instructions that could not work for that provider. Other providers could report a working configuration that had never been validated. Guidance is now tied to the provider itself, and an unrecognized provider says so plainly instead of borrowing advice from a similar one.
- **`--help` on a launch command shows help instead of starting work.** Asking a launch command for help previously began launching, which on a restricted or read-only cache directory failed with a permissions error and no help at all. Help is now shown immediately and changes nothing. To send `--help` to the tool being launched rather than to NeuroRouter, put `--` before it. A launch whose cache directory cannot be written also now continues with a clear notice instead of stopping.
- **Startup and `doctor` report the same result every run.** When more than one provider was configured, the sign-in compatibility check could report different severities on identical configuration between runs. It now consistently reports the most serious finding.

## [0.36.9] - 2026-08-08

### Fixed
- **Reasoning models are no longer intermittently blocked on the Anthropic endpoint.** Some reasoning responses were occasionally refused when secret protection was active — roughly one request in ten on reasoning-heavy traffic. Those responses now pass through normally, while genuinely protected provider content is still preserved exactly.
- **Provider URLs containing a username or password are refused before the proxy starts.** Previously such a URL was only rejected once a request was made. The startup banner also no longer prints credentials in a target address, while still showing the host and path you configured.
- **A mistyped provider endpoint is flagged at startup.** When the proxy recognizes the provider you are pointing at, it now tells you up front if the address does not match that provider's expected endpoint for the protocol you selected, instead of failing later with an unclear error. Unrecognized providers are left alone.

## [0.36.8] - 2026-08-07

### Fixed
- **Requests that mention placeholder syntax in their content are no longer blocked.** Secret protection could mistake ordinary text that describes a redaction placeholder for a real unresolved secret and refuse the request. It now recognizes only genuine placeholders, so normal content passes through while real secrets stay protected.

## [0.36.7] - 2026-08-07

### Fixed
- **Reasoning models now proxy reliably.** Responses from models that emit reasoning output are no longer occasionally blocked when secret protection is active, so routine requests to these models go through as expected while sensitive content stays protected.
- **Context-overflow protection now covers every launch path.** The safeguard that keeps a request from exceeding the model's context window now applies even on direct passthrough sessions, so a large or resumed session is stopped with a clear message before it can overflow instead of failing partway through.

### Changed
- **A refused oversized request no longer affects provider availability.** When a request is declined locally for being too large, that decision no longer counts against the provider's health, so an unrelated request is never held back because of it. Genuine provider errors are still tracked as before.

### Fixed
- **Context-overflow protection now applies consistently to every launched tool and every provider.** The safeguard that keeps a request from exceeding the model's context window is now enforced on one shared path, so it works the same way regardless of which agent or provider you use. When the proxy can't determine a model's context limit, it now protects you with a conservative limit and tells you, instead of quietly doing nothing — preventing a session from silently growing until it fails.
- **More reliable request routing to providers with versioned endpoints.** Requests to providers whose API URL includes a version in its path are now routed correctly across all request types, and any query parameters or path details in your configured URL are preserved.

### Security
- **Provider endpoint URLs with embedded credentials are rejected.** If a configured provider URL contains a username or password in the address itself, the proxy now refuses it instead of forwarding it.

## [0.36.5] - 2026-08-04

### Fixed
- **Proxying to OpenAI-compatible providers with a versioned endpoint now works.** When you point the proxy at a provider whose API URL already includes a version in its path, chat and response requests now reach the provider correctly instead of failing with a "not found" error. Standard OpenAI endpoints are unaffected, and any query parameters or path details in your configured URL are preserved.

## [0.36.4] - 2026-08-03

### Fixed
- **Model discovery now works when proxying an Anthropic endpoint.** Asking the proxy for the list of available models now returns the upstream provider's catalog instead of a "not found" error, using the same authentication as your normal requests. If the provider can't be reached, the request fails clearly rather than returning an unrelated list.

## [0.36.3] - 2026-08-02

### Fixed
- **Hivebus version skew no longer floods proxy logs.** When another local session requires a newer hivebus protocol, NeuroRouter reports the condition once and stops futile background retries until normal request activity proves the peer is compatible again. A dedicated setting can lower or silence these heartbeat-only notices without hiding other proxy errors.

## [0.36.2] - 2026-08-02

### Fixed
- **More reliable resource-pressure warnings on macOS.** The proxy now reports its open-connection usage accurately on macOS, so it can warn you before running low on system resources instead of failing silently.
- **The proxy recovers from an internal fault instead of going quiet.** If the proxy's request handler hits an unexpected internal error, it now restarts itself and keeps serving — or, if the fault persists, exits cleanly so your supervisor restarts it — rather than staying up but unresponsive.

## [0.36.1] - 2026-08-01

### Fixed
- **Clearer, safer proxy authentication.** When you configure the proxy to use its own API key but that key isn't available, the proxy now stops with a clear message at startup instead of quietly falling back to forwarding the caller's credentials. Each provider uses only its own key, and authentication behaves the same way across providers.

## [0.36.0] - 2026-07-31

### Added
- **Stronger cache preservation across all supported providers.** NeuroRouter works with a warm provider cache instead of against it, so context management preserves what the provider is already caching. This applies both when you launch a tool through NeuroRouter and when you route your own software through it as a plain proxy.
- **Configurable context-management policy for every provider.** A per-deployment setting selects how aggressively context is managed (`off`, `cache_safe`, `balanced`, `aggressive`), defaulting to the cache-safe level, with the same choices available for each provider.
- **A read-only status view of the current context-management posture.** See, at a glance, how each provider is configured. The view is local and contains no request content.

### Changed
- **Context-overflow protection is always on by default.** Oversized tool output and superseded results are trimmed so a request cannot overflow the model's context window — this protection stays on even when the cache state is unknown. Setting the policy to `off` respects your explicit choice to disable it.

### Fixed
- **The audit view no longer contains raw request text.** Free-text fields on the served audit surface are represented by digests; full local detail remains available for your own diagnostics.
- **A rare proxy error on session resume is fixed.** Requests that forked a session with a stale in-flight tool call could previously fail; they are now handled cleanly.
- **Improved shutdown reliability on Windows.** A rare condition that could cause the proxy to hang during shutdown on Windows has been resolved.

## [0.35.5] - 2026-07-27

### Changed
- Launches are quieter: NeuroRouter no longer shows a "local path exposure(s) detected" notice on every launch. File paths are not credentials, so this is now an optional, verbose-only note rather than an every-launch message.

## [0.35.4] - 2026-07-27

### Added
- `nr metrics` shows rolling cache-read percentages for each connected agent and provider using NeuroRouter's own local, content-free usage records. When a no-shaping baseline has not been measured, the report says the baseline and delta are unknown instead of implying savings it cannot prove.

### Fixed
- Launched Codex sessions can safely forward structured tool results through request protection without an unexpected blocked-request error or a change to the tool-result shape.
- NeuroRouter now surfaces when it falls back to a legacy configuration directory instead of silently using two config locations, so a stuck configuration migration is visible rather than hidden.

## [0.35.3] - 2026-07-26

### Fixed
- Launched sessions that switch models mid-conversation (for example, moving to a newer model) no longer fail with a request error. Conversations continue cleanly across the switch while request-safety checks remain in force.

## [0.35.2] - 2026-07-26

### Security
- Secret-like code references and examples now remain intact when NeuroRouter cannot confirm they are credentials. Validated built-in credentials and values you explicitly configure remain protected before model traffic leaves the machine, including older Vault credentials supplied through the standard environment-variable assignment.

### Fixed
- Windows-formatted (CRLF) private keys and AWS access keys containing digits are reliably protected.

## [0.35.1] - 2026-07-24

### Fixed
- Built-in provider tools such as web search continue to work in launched Codex and Claude sessions without weakening protection for client tool output. The same boundary now applies to HTTP and Responses WebSocket traffic, while hosted and public-bound deployments retain the blocking posture.
- Launched Codex sessions now reliably route their traffic through NeuroRouter for both supported sign-in methods, so protection and accounting apply consistently. Your existing Claude launches are unaffected.

## [0.35.0] - 2026-07-23

### Added
- Qwen Code launches through NeuroRouter using the credentials it already has.
  If you run `nr launch qwen` without exporting a provider key, NeuroRouter now
  keeps Qwen Code's own working setup instead of refusing to start. Set an
  explicit `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` and that still takes precedence.
- Provider-native tools work through the launcher. Codex and Claude launches can
  now use built-in tools like web search without hitting an unexpected error;
  hosted and public-bound launches keep their existing protections.
- Experimental dense cache mode: NeuroRouter can compact eligible older
  conversation text after protection runs, preserving the active task and the
  provider's own cache boundaries. It never claims a cache hit it can't prove, and
  privacy/secret protection always runs first.

### Fixed
- More accurate cost accounting under load. Concurrent streams no longer pick up a
  neighbouring request's pricing tier, and requests without cache markers are no
  longer billed as if cached. Behaviour for marked cache requests is unchanged.
- Streaming protection is steadier: a protected value split across stream chunks is
  reassembled before it is restored, so partial values are never emitted.

### Security
- Provider-signed and encrypted response data (reasoning, compaction, and similar
  opaque fields) is preserved exactly through protection on every transport; when a
  rewrite would be ambiguous, NeuroRouter fails closed rather than risk corrupting a
  signed payload. High-confidence secrets stay protected even under warning-only
  policy, and separate tenants/sessions can no longer collide.

## [0.34.0] - 2026-07-21

### Added
- You can now choose how Qwen Code connects: `nr launch qwen --auth-type anthropic`
  or `--auth-type openai` selects the wire explicitly, and NeuroRouter checks it is
  consistent before starting — a mismatched target is refused up front instead of
  failing partway through a session.
- Qwen sessions keep their provider-side cache. When NeuroRouter shapes a Qwen
  conversation, it preserves the response chain the provider uses for its own
  caching, so repeated turns keep hitting that cache instead of restarting it.
  Privacy and secret redaction still always take precedence — a protected value is
  never re-exposed to win a cache hit.

### Fixed
- Hardened concurrency on the Qwen streaming path: under heavy concurrent use, an
  internal audit record could be read while it was still being updated. NeuroRouter
  now takes a stable snapshot before recording, removing an intermittent race that
  could surface under load. No configuration or behavior change for operators.

### Docs
- New guide: running a harness secret scanner alongside NeuroRouter. Explains the
  two separate control points — a harness-side scanner (pastewatch is our free
  reference integration) covers secrets in local files, pastes, and tool output
  before they reach a model; NeuroRouter covers secrets at the model/provider
  traffic boundary. Neither replaces the other. Includes how to keep an existing
  harness scanner (e.g. GitGuardian ggshield's AI coding-tool hooks) and add
  NeuroRouter at the provider boundary.

## [0.33.1] - 2026-07-21

### Fixed
- Launching Qwen Code with `nr launch qwen` works again. It was failing to start with
  an internal identity-mode error; it now launches normally.

## [0.33.0] - 2026-07-21

### Fixed
- NeuroRouter now preserves your provider's prompt cache. When your request marks a
  cacheable prefix, NeuroRouter keeps those exact bytes untouched as it shapes the rest
  of the conversation, so your cached prefix keeps hitting the provider's cache instead
  of being invalidated — which matters most on plans where cache hits drive your cost.
  Privacy and secret redaction still always take precedence: if protecting a value
  requires changing it, that change is kept and the cache break is recorded, never
  silently reverted to win a cache hit. A debug override is available to turn this off
  for troubleshooting.

## [0.32.0] - 2026-07-21

### Fixed
- The list of your live agent sessions is now accurate. Previously, after restarting
  your machine (or across long-running work), the boardroom roster could show sessions
  that were no longer running — ghosts — and the same session could appear under a
  shifting name. NeuroRouter now treats a session as live only when the process behind
  it is genuinely running, so restarting clears out stale entries automatically, the
  count matches the sessions you actually have, and each session keeps a stable address
  you can reliably message.
- Messages between your sessions now send immediately. An announcement or question from
  one session used to wait until that session next did something before it was delivered;
  now it goes out right away, so a session that asks a question and then sits idle still
  reaches the others.

## [0.31.0] - 2026-07-21

### Added
- A single oversized file read or paste can no longer wedge your session. NeuroRouter
  now stops any input that would not fit your model's context window before it reaches
  the conversation — reading a huge file, pasting a large blob, or a tool returning a
  giant result is trimmed up front, with a clear marker left in place of the oversized
  content so the model knows it was shortened (not corrupted). This applies to file
  reads, pastes, and results from web fetches, connected tools, and sub-agents. When
  there is genuinely no room left, NeuroRouter refuses cleanly with guidance instead of
  letting the session get stuck in an unrecoverable state. The behavior turns on
  automatically based on your provider's context window.

## [0.30.0] - 2026-07-20

### Added
- NeuroRouter now keeps a private, on-machine record of the effect-bearing actions
  your launched agent sessions take (things like commits, pushes, releases, and
  file edits) as it observes them — recording only the action type, the tool, the
  session, and a timestamp. It never records the contents of those actions and never
  stores secrets, the log stays local to your machine, and it is strictly a
  measurement: it observes and never blocks or changes what your agents do. This
  gives you an honest, auditable picture of how much of your agents' activity your
  local governance layer actually sees.

## [0.29.0] - 2026-07-19

### Added
- Your local agent sessions can now hold a real back-and-forth on the same machine.
  One session can ask another a question and get an answer back, delivered as a short
  routing note in the conversation while the actual question and answer stay on the
  authenticated inbox. Announcements and questions are now clearly distinct (an
  announcement has no reply path), and you can page through recent boardroom history
  with time and count limits.

### Security
- Answers between your sessions are bound to the exact peer they were meant for and
  to their verified signature. A signed answer replayed on a different channel, or one
  whose sender is spoofed, is rejected instead of being shown to your agent.

## [0.28.1] - 2026-07-19

### Fixed
- Launching Qwen Code with OAuth now gives you a clear, accurate message instead
  of a misleading one. Qwen's own OAuth sign-in was retired upstream in April 2026,
  so NeuroRouter supports Qwen through your Model Studio API key — and now says so,
  telling you exactly how to connect (and to use Qwen directly if you want the
  Alibaba Cloud Coding Plan). Nothing about the existing Model Studio key setup
  changes.

## [0.28.0] - 2026-07-19

### Added
- You can now rotate NeuroRouter's machine-local hivebus credentials in place,
  without restarting your running sessions. If a credential is ever exposed, a
  single rotate command issues fresh credentials, running sessions pick them up
  on their own, and the old credential stops working — no fleet-wide restart
  required. Rotation is atomic on macOS, Linux, and Windows, and the credential
  a session presents cannot be forged or replayed.

### Changed
- Launched sessions no longer carry a copy of the raw machine credential in their
  environment. They hold a reference and read the live credential on demand, so a
  rotation reaches them automatically and an exposed credential affects far less.

### Fixed
- Directives sent to your own machine's local hivebus work again without extra
  configuration; only genuinely remote (Teams / dedicated-server) endpoints
  require an explicit directive-authority setting.

## [0.27.1] - 2026-07-19

### Fixed
- NeuroRouter no longer blocks a completion because it thinks your work is in the
  "wrong" place. Previously, finishing a task from a valid working copy or branch
  that did not match NeuroRouter's guessed expectation could be held up entirely.
  Now those checks advise rather than block: NeuroRouter surfaces a note about what
  it observed and lets your work proceed — it does not overrule where you chose to
  do your work. Checks that catch a provable, concrete problem still apply
  (for example, claiming a task done while changes are still uncommitted).

## [0.27.0] - 2026-07-18

### Added
- Delegated agent sessions are now governed before they can change anything. When
  a supervised Claude Code or Codex session spawns a child agent, that child can
  no longer commit, push, publish a release, edit files, run mutating shell
  commands, or write to your work-order system unless it carries verified
  authority for that exact action. Read-only work (searching, viewing, inspecting)
  continues to flow freely. Attempts to turn the protection off from the client
  side are rejected.
- The protection is safe-by-default: if a child agent's authority cannot be
  verified, the action is blocked rather than allowed, and the block is reported
  with a clear reason and how to proceed (import trusted evidence, escalate
  authority, or run under the parent's claim). Actions from tools that run outside
  the supervised launch are not covered by this guarantee and are documented as
  out of scope.

## [0.26.1] - 2026-07-17

### Added
- Qwen Code, OpenCode, and other OpenAI-compatible chat clients can now connect
  through NeuroRouter using the standard `/v1/chat/completions` endpoint. Your
  provider's own request and streaming fields pass through unchanged, streaming
  works end to end, and the same secret protection and credential handling you get
  on the other routes apply here too. If your target model can't do something the
  request needs (for example tool calls or structured JSON output), the request is
  refused up front with a clear error instead of being sent and failing silently.

### Fixed
- Long agent turns no longer get cut off. Previously a request that streamed for
  a long time — common with large contexts — could be dropped by an internal
  timeout, forcing the whole turn to be redone. NeuroRouter now keeps a streaming
  response alive as long as it is actively producing output, and only ends it if
  the upstream truly goes silent. Short requests are unaffected.
- An internal safety check no longer blocks legitimate requests. It could
  previously stop a request from being sent when your prompt discussed security
  topics; it now forwards the request unchanged and simply notes the observation,
  so normal work is never interrupted.
- Concurrent spend and usage tracking no longer drops records under heavy parallel
  tool use. Previously, bursts of simultaneous requests could hit a brief database
  lock and silently lose a usage record, skewing your cumulative stats; NeuroRouter
  now waits out the contention on every connection, so your accounting stays
  accurate. (Also resolves an intermittent failure seen only on Windows.)
- Peer coordination messages between your local sessions are now verified before
  they can reach a model. A coordination post is delivered into a session only if
  its signature is intact, so a tampered or unsigned message cannot be surfaced to
  your agent; resumed sessions now receive these posts too, not just fresh ones.

## [0.26.0] - 2026-07-14

### Added
- Run multiple local proxies without collisions, and see what is running. New
  commands `nr proxies` and `nr runs` list every proxy on this machine with its
  address, process, and whether it is still alive; `nr cleanup` reclaims proxies
  that have exited (with `--dry-run` to preview and an opt-in `--purge-captures`).
  A second proxy started on the default port now picks a free port automatically
  instead of failing, while an explicit port conflict still fails fast and tells
  you which proxy holds it. `nr metrics` finds a running proxy on its own when you
  do not pass `--addr`. Works on macOS, Linux, and Windows.

### Improved
- Context shaping now tidies each turn as it enters the model, so long sessions
  send a cleaner, more consistent context. This is a conservative first pass that
  preserves your content, applies only when shaping is on, and is designed to keep
  the parts that let caching work unchanged — so it should never make a request
  cost more. Turn it off with `shaping = 0` (pass-through) if you prefer.

### Changed
- NeuroRouter now stores its logs and internal approval state under the standard
  location `~/.local/state/neurorouter/` (following `XDG_STATE_HOME`), keeping them
  separate from your configuration in `~/.config/neurorouter/`. If you had data in
  the older `~/.neurorouter/` location it is migrated automatically and safely on
  first run; your existing files are never overwritten, and your prior logs remain
  readable where they were.

## [0.25.4] - 2026-07-11

### Added
- Support for the latest OpenAI models (GPT-5.6 Sol/Terra/Luna and GPT-5.3 Codex),
  including their reasoning-effort options and long-context pricing. Model details
  ship as editable data, so you can add or correct a model yourself without waiting
  for a new build and with no outbound network calls — point
  `NEUROROUTER_MODEL_CATALOG_JSON` at your own catalog file to override.

### Fixed
- Safer model switching mid-session: if a session switches models while a tool
  call is still open, NeuroRouter now returns a clear error asking you to start a
  new session, instead of silently continuing with mismatched state and losing the
  tool result. When a model's price isn't published yet, it is reported as unknown
  rather than shown with a guessed price.

### Added
- Smoother behavior under load across multiple sessions: NeuroRouter now tracks
  upstream rate limits and provider cooldowns in shared local state, so parallel
  sessions back off together instead of repeatedly hammering a provider that is
  already rate-limiting, and a cooldown is remembered across a restart. Upstream
  rate and cooldown summaries are visible in `nr metrics`.

### Fixed
- Long-running proxy sessions are more stable: NeuroRouter no longer leaks
  system resources per request or lets its local state file grow without bound,
  two issues that could slow down and eventually stop the proxy during extended
  use. Existing state files are cleaned up automatically on the next start,
  after the proxy is already accepting requests, and stale entries are trimmed
  in the background as the proxy runs.

### Changed
- NeuroRouter now writes a local, rotation-bounded, secret-redacted proxy log by
  default at `~/.neurorouter/logs/proxy.log` so non-panic proxy failures can be
  diagnosed after the fact. Set `log_file_enabled = false` to disable this
  durable proxy log; panic crash logs remain separate.

## [0.25.2] - 2026-07-09

### Fixed
- Internal test-reliability fix; no change to runtime behavior. A routine follow-up to the 0.25.x line.

## [0.25.1] - 2026-07-09

### Fixed
- Internal reliability fix for NeuroRouter's own build and test tooling on Linux hosts. No change to runtime behavior; a routine follow-up to the 0.25.0 release.

## [0.25.0] - 2026-07-09

### Added
- You can now wait for a reply when one local session asks another a question: `nr hivebus ask --wait-seconds=N` (and `nr hivebus call --wait-seconds=N` for a broadcast) sends the question and waits up to the given window for an answer, then reports clearly if none arrives. Without `--wait-seconds` the behavior is unchanged — the question is sent and the command returns immediately.

### Fixed
- A waited-for answer is now trusted only if it genuinely came from the peer you asked: replies are cryptographically verified and, for a directed question, bound to the session and participant you addressed — so another local session cannot answer in someone else's name.
- Codex now works with web search enabled through NeuroRouter — the earlier "tool output protection blocked" error when Codex declared its built-in web search tool is resolved for your own signed-in Codex sessions. Multi-tenant/hosted and public-bind deployments keep the protection.
- Multi-line messages between local sessions are delivered whole instead of being cut off at the first line break.

## [0.24.25] - 2026-07-08

### Fixed
- Codex now completes real tool-using work through NeuroRouter on your ChatGPT sign-in — the earlier "tool calling is not available on this route" error on `nr launch codex` is resolved.
- NeuroRouter now recovers from transient DNS failures on its own: a brief network blip or reconnect no longer leaves the proxy stuck returning "no such host" on a working connection.
- The proxy survives a crash without losing the reason: crashes are recorded to a durable local log so a failure is diagnosable instead of silent.
- A renewed license is picked up by the running proxy without a restart.
- Peer coordination between your local sessions rides through a brief host handoff instead of dropping to solo the instant the hosting session restarts.

### Changed
- Hardened release and completion verification so a red or incomplete CI run cannot be mistaken for a green one.

## [0.24.24] - 2026-07-06

### Fixed
- Codex can now run through NeuroRouter using your ChatGPT sign-in, without requiring an OpenAI API key.
- Updated Codex guidance to distinguish the supported ChatGPT backend relay from the OpenAI platform API path, which still requires API-key access.

## [0.24.23] - 2026-07-04

### Fixed
- The secret guard no longer alters your output when it is only guessing. NeuroRouter now
  changes what the model sees only for a secret it can actually confirm — a license key
  that verifies cryptographically, or a value you have listed as a secret in your config.
  Anything it merely suspects (a token that happens to look key-shaped, a `gcloud
  --update-secrets` line that references secrets by name, or ordinary text that trips a
  pattern) is reported to you out-of-band and left in your output exactly as written, so
  your commands stay intact and copy-pastable. When it flags something, it tells you the
  one config line to add if it really is a secret you want protected.

### Changed
- All of NeuroRouter's configuration and local state now live in one place —
  `~/.config/neurorouter` (following `XDG_CONFIG_HOME`) on every platform — instead of
  being spread across a few different folders. If you have existing files in the old
  locations, they are copied over automatically and safely on first use (nothing is
  deleted or overwritten, and file permissions are tightened to owner-only during the
  move). No action needed on your part.

## [0.24.22] - 2026-07-04

### Fixed
- Replying to a session message now works even after that session restarts. When you
  answer a message addressed to a session by its handle, NeuroRouter finds the handle's
  current live session — so your reply lands instead of failing with "not found." This
  completes reliable two-way session-to-session messaging.
- Secret warnings no longer disturb your command output. When NeuroRouter is only
  *guessing* that something looks like a secret (for example a `gcloud --update-secrets`
  line that references secrets by name), it now tells you separately instead of writing
  a warning into the middle of the text — so the command stays intact and copy-pastable.
  A confirmed, known-format secret is still flagged the usual way.
- A dropped connection no longer freezes your session. If a streaming request loses its
  connection mid-response, NeuroRouter now clears the stale connection so your next
  request just works — previously you had to restart the session.
- On Claude models with the 1-million-token context option, the context meter shows the
  right percentage instead of reading as "full" too early.

### Added
- Cost figures are labeled honestly: where NeuroRouter shows a dollar amount it now says
  how confident that number is (measured vs. estimated) and leads with token counts, so
  an estimate is never shown as if it were a measured charge. Starting on the
  smoke-test/cost-report surfaces.

## [0.24.21] - 2026-07-03

### Fixed
- Messaging a specific session by its handle is now reliable, even after that
  session restarts. When you send to a session by its stable handle, NeuroRouter
  now figures out the session's current identity at send time — so a message
  reaches the session that's live right now instead of getting stuck on an old,
  restarted copy. Sending to the room, and operator directives, work exactly as
  before. This is the last piece that makes same-machine session-to-session
  messaging dependable end to end.
- On Claude models running with the 1-million-token context option, the context
  meter now shows the right percentage instead of reading as "full" too early.

### Added
- Cost figures are now labeled honestly. Where NeuroRouter shows a dollar amount
  it now says how confident that number is (measured vs. estimated) and leads with
  token counts, so an estimate is never shown as if it were a measured charge. This
  starts on the smoke-test/cost-report surfaces and will expand to more surfaces.

## [0.24.20] - 2026-07-02

### Fixed
- Reading your boardroom is now reliable. `nr boardroom inbox` shows the shared
  boardroom log and marks which messages are for you — directly addressed, likely
  meant for you (matched to your role/handle even after a restart), addressed to a
  session that is no longer current, or broadcast to everyone. It no longer hides a
  message just because your session restarted and got a new internal id, and it no
  longer needs a special launch to read. This fixes the case where a question sent
  to you showed up as "no asks" even though it was on the board.
- Boardroom sessions on macOS and Linux can now recover from an older local host
  while the system is already running. When one session has upgraded, it can take
  over the shared local boardroom host and reconnect its boardroom client without
  you manually restarting the session that happened to start first.
- Stopping an older preempted boardroom host no longer removes the newer host's
  local socket on macOS and Linux.
- On Windows, `nr hivebus status` now states the named-pipe handoff limitation
  explicitly: a live older local host cannot be force-replaced until a cooperative
  handoff release exists.

## [0.24.19] - 2026-07-02

### Fixed
- Fixed a regression that could make every request to a Sonnet model fail. Recent
  Claude Code versions send an extended-thinking cleanup setting alongside thinking;
  NeuroRouter removed the thinking part but left the cleanup setting behind, so the
  provider rejected the request. NeuroRouter now keeps the two in sync — if it has to
  drop thinking, it also drops the dependent cleanup setting while leaving your other
  settings intact — so your Sonnet requests work again. Upgrade to 0.24.19 if you saw
  a "clear_thinking" or thinking-related request error on 0.24.18.

## [0.24.18] - 2026-07-02

### Added
- Claude Sonnet 5 support. NeuroRouter now recognizes Sonnet 5, gives it the right
  context window, and automatically drops request settings Sonnet 5 no longer
  accepts (such as temperature and manual thinking budgets) so your requests keep
  working instead of erroring. Sonnet 4.6 is unaffected, and token and cost
  estimates account for Sonnet 5's updated tokenizer and pricing. The same handling
  applies to Opus 4.7 and 4.8.

### Changed
- Boardroom messages between your sessions are now more reliable. A message stays
  readable until it expires instead of disappearing once delivered, so answering a
  question you received works even after you've read it, your inbox shows both
  broadcasts and direct questions in one place, and messages still reach a session
  after it restarts or its display name changes. Sessions on one machine can also
  notice relevant messages that weren't addressed only to them. Operator directives
  are unchanged — they still go only to their intended session.

## [0.24.17] - 2026-07-01

### Fixed
- Your same-machine sessions now reliably see each other in the boardroom. Before,
  sessions started in separate sandboxes could each end up on their own local group
  and never share messages — so `nr boardroom who`, `ask`, and `call` appeared to
  work but traffic never crossed between them. Sessions on one machine now join the
  same local group regardless of how they were launched, and stale leftovers from
  closed sessions are cleaned up automatically.

## [0.24.16] - 2026-06-30

### Added
- `nr boardroom call "<question>"` lets one session ask the room a question and have
  the right peer answer, instead of you having to know which session to message
  first. Add `--role`, `--repo`, or `--capability` to aim the question at a kind of
  session (for example, the architect working in a given repo). Once a peer answers,
  you continue the conversation directly with `nr boardroom ask <handle>`.
- When a session receives a directed question, the message now includes the exact
  command to reply with, so it can answer on its own without being told how.

### Fixed
- A launched session now reliably keeps a stable boardroom identity, and reading your
  inbox no longer adds a phantom entry to the roster.
- The safeguard that caps oversized tool output now counts tokens conservatively for
  dense content (code, JSON, and CJK text), so a large result is trimmed instead of
  slipping past the cap. Normal context-window handling is unchanged.

## [0.24.15] - 2026-06-29

### Added
- Boardroom sessions now have a short, readable handle like
  `architect/neurorouter-pro#a1c2`, shown as the first column of `nr boardroom who`.
  You can address a session by that handle instead of a long opaque id —
  `nr boardroom ask architect/neurorouter-pro#a1c2 "..."` — and a new
  `nr boardroom find <role> <repo>` looks up the handle(s) for a role in a repo, so
  one session can find and message another (for example, a worker finding its
  architect) without hunting through the roster. Handles stay stable across
  restarts and remain visible even when an idle session's model/repo details age
  off the card.

## [0.24.14] - 2026-06-29

### Fixed
- Reading your boardroom inbox no longer clutters the roster. Before, checking the
  inbox repeatedly (for example, polling it on a timer) could fill `nr boardroom who`
  with extra "architect" entries that weren't real sessions — and that, in turn,
  could make asking a session by name ambiguous. Inbox reads are now passive and add
  nothing to the roster.
- A session's boardroom card now shows the repository it's actually working in,
  instead of sometimes showing the wrong one.

### Changed
- The safeguard that stops a request from exceeding the model's context window now
  works correctly for every provider — Claude, OpenAI, and Qwen — and applies a safe
  conservative limit to any model it doesn't recognize, instead of turning the
  safeguard off.

## [0.24.13] - 2026-06-28

### Fixed
- Asking another session a question now works reliably. `nr hivebus ask <session>`
  resolves who you're asking at the moment you send, so a session that shows as live
  in `nr boardroom who` can be reached instead of being reported as "not
  addressable." If a target can't be found, the error now lists the live session ids
  you can actually use.
- The boardroom roster keeps idle sessions visible without showing stale details.
  An idle session stays on the roster, but its model and repository are no longer
  shown once that information is old — so you don't see a frozen, possibly-wrong
  label for a session that has been sitting idle.
- Restarting a session now cleanly replaces its old roster entry instead of leaving
  both the old and new card visible.

## [0.24.12] - 2026-06-27

### Fixed
- `nr boardroom who` now reliably shows the sessions you have running, even when
  they are idle. Before, a session that wasn't actively sending requests could drop
  off the roster between requests — so `who` could briefly show "no observed
  sessions" while your sessions were still running — and then reappear on the next
  request. Idle sessions now stay visible. Presence stays honest: a session that
  crashes still drops off on its own, and one you close leaves right away instead
  of lingering.

## [0.24.11] - 2026-06-27

### Fixed
- Resumed Claude sessions now keep the model you are actually running. Previously,
  resuming a session whose history included earlier Opus turns could cause the
  resumed session to be detected as Opus and routed to Opus 1M — even when you were
  on Sonnet — which both mislabeled the session and could increase cost. Resumed
  sessions now use the most recent model from the transcript, so Sonnet stays Sonnet
  and a genuine Opus resume still gets the 1M context window.

### Fixed
- The local boardroom roster now shows the sessions you actually have running.
  Before, `nr boardroom who` could fill with stale or mislabeled entries (wrong
  repo or model) while real sessions were missing or shown twice. Sessions now
  appear with correct details, a restarted session replaces its old entry instead
  of showing both, and a session only joins the roster once it has a real identity
  from the launcher. Sending and listening on the boardroom are unchanged.

### Fixed
- Resumed sessions now show up in the local boardroom. Previously, a session
  started with resume/continue would run normally but never appear in
  `nr boardroom who`, so long-running workers were invisible to other
  same-machine sessions and restarting did not help. They now register on their
  first request. Sessions you have muted (go-dark) still stay hidden.

## [0.24.8] - 2026-06-26

### Added
- Local boardroom speakers that don't carry their own session id — such as Codex
  and headless agent runs — now show a clear, stable name per repository, like
  `neurorouter-pro/codex-01` and `neurorouter-pro/codex-02`, so you can tell several
  same-kind agents apart in one room instead of seeing one shared label. Names are
  assigned locally on your machine and reused as agents come and go. Sessions that
  already have their own id keep it. This is a display name for telling agents apart,
  not a security or authentication claim.
- Added `nr boardroom` and `nr localbus` as clearer names for local same-machine
  boardroom messaging. `nr boardroom say` posts attributed messages, and
  `nr boardroom listen --json` reads the same non-consuming local thread. The
  existing `nr hivebus` command still works as a compatibility alias.

## [0.24.7] - 2026-06-24

### Fixed
- Fixed another local coordination split on macOS/Linux. Sessions launched from
  different sandboxed terminals now still join the same same-machine group, so
  boardroom messages and fleet visibility work across those sessions after all
  processes restart onto this version.

## [0.24.6] - 2026-06-23

### Fixed
- Fixed a regression where your sessions could split into separate groups that
  could not see each other. Local coordination now uses the operating system's
  same-machine channel directly, so all of your sessions on one machine reliably
  join the same local group again, including on Windows.

## [0.24.5] - 2026-06-23

### Fixed
- Fixed a regression where your sessions could split into separate groups that
  could not see each other. All of your sessions on one machine now reliably join
  the same local group again.

## [0.24.4] - 2026-06-23

### Changed
- Same-machine session coordination is now rock-solid. The local coordination now
  uses a fixed, computed port instead of a discovered one, so your sessions
  reliably find each other, recover instantly if the hosting session closes, and no
  longer drop into a state where each session sees the bus but not the others. No
  setup, no server — it just works and stays working.

## [0.24.3] - 2026-06-23

### Fixed
- Same-machine session coordination now recovers on its own. If the session that
  was hosting the local coordination closes, another running session automatically
  takes over — so your sessions keep seeing each other instead of silently dropping
  to solo until you start a new one.

## [0.24.2] - 2026-06-23

### Fixed
- Resuming a session that can't be restored cleanly no longer fails to launch.
  When a session falls back to its preserved brief, the launcher now starts the
  fresh session reliably instead of exiting with an error.

## [0.24.1] - 2026-06-23

### Fixed
- `nr hivebus status` now correctly reports same-machine coordination as the
  local Pro mode. A normal multi-session setup was being mislabeled as a
  dedicated team server, and a leftover address from a closed session could make
  a perfectly healthy setup look offline. Status now reflects what's actually
  running.

## [0.24.0] - 2026-06-22

Your same-machine sessions can now actually talk in their work — and a new pipe
turns a wall of mess into a crisp answer.

### Added
- **The boardroom reaches your agents.** When you run more than one session, a
  participating session now sees the room's messages surface in its own turn — so
  agents coordinate without you relaying. Each message shows whether it's one the
  agent may reply to or just observe, and a session is told once how to take part
  (it's always your choice — a quiet/private posture stays out). `nr hivebus
  status` shows whether a session is participating.
- **`nr distill` — mess in, crisp answer out.** Pipe a huge log or dump plus a
  question and get a short, accurate answer without pasting it all into a session:
  `cat huge.log | nr distill "why did the build fail?"`. The answer is a
  reduction, not the whole source — and your input is secret-scrubbed before it
  ever leaves the machine.
- **Turn a useful exchange into a tracked item.** `nr hivebus promote` takes a
  boardroom message and opens a work order from it, with a link back to the
  conversation — so a good idea from the room doesn't evaporate when the session
  ends.
- **Sessions find each other instantly.** A newly launched session now appears to
  the others right away, without having to send a request first.

### Changed
- Coordination stays disciplined by design: the boardroom is for signals (hints,
  priorities, findings), it's ephemeral, and anything that must last is meant to
  become a tracked work item — your sessions are told this when they join.

### Fixed
- Hardened first-time setup so two sessions starting at the exact same moment can
  never trip over each other's credentials file.

Your same-machine sessions can now hold a conversation in one shared room.

### Added
- **A boardroom for your sessions.** `nr hivebus say "<message>"` speaks to every
  other NeuroRouter session on your machine at once, and `nr hivebus listen` shows
  the running thread — anyone who knows can answer. There is no need to name who you
  are talking to: you speak to the room, not to one session. Speaking adds nothing to
  `nr hivebus who` (no stray rows), and every session sees every message until it
  ages out. It stays entirely on your machine, like everything else in Pro.

### Changed
- If environment variables are pointing NeuroRouter at a dedicated team server,
  `nr hivebus status` and startup now say so plainly — naming which variables are in
  effect (never their values) and how to return to the single-machine setup — so you
  always know which mode you are in.

## [0.22.1] - 2026-06-21

A fix so your same-machine sessions can actually reach each other.

### Fixed
- Local sessions now show as live and addressable in `nr hivebus who`, and
  `nr hivebus ask` reaches them. A timing-precision mismatch in the presence
  signature was causing every session to be shown as unverified and
  unreachable, so asking another session could not find it. Sessions you can see
  are now sessions you can message.

## [0.22.0] - 2026-06-21

Your agents can now talk to each other — and the conversation never leaves your
machine.

When you run more than one NeuroRouter-fed agent session on the same computer,
they form a local mesh: each one can see the others and you can hand a directive
from one to another. There is **no server to run, nothing to configure, and no
network traffic** — the coordination happens entirely between the NeuroRouter
processes on your own machine, over a local-only channel that is refused the
moment anything tries to point it off-box. NeuroRouter is talking to NeuroRouter;
your code, prompts, and agent activity stay on your laptop.

(Connecting agents **across** machines or a whole team is a NeuroRouter
Team/Enterprise capability — a dedicated, governed server that crosses the machine
boundary on purpose. Pro is deliberately one machine, by construction.)

### Added
- **Same-machine agent mesh, zero setup.** Start two or more sessions and they
  find each other automatically — the first one hosts the local hive, the rest
  join it. No server, no environment variables, no manual keys. If only one
  session is running, it simply works solo.
- **See your fleet:** `nr who` / `nr fleet` show the other local sessions (model,
  repository, current task), and `nr hivebus status` shows whether the local hive
  is up, your privacy posture, and how many sessions are connected — without ever
  printing secrets.
- **Hand off work:** `nr hivebus '[hb] <agent> <task>'` delivers a signed request
  to another local session. It arrives as a message that session can act on — it
  is never injected into the agent's output, and a session only accepts a
  directive that carries your local operator authority.
- **Sessions can ask each other.** `nr hivebus ask <agent> <question>` lets one
  session ask another a question directly; `nr hivebus inbox` shows questions
  addressed to you and `nr hivebus answer` replies. An ask is just a question —
  the other session chooses whether to answer, and an ask can never make a peer
  act. This is how your sessions coordinate without routing every message through
  you.
- **Go quiet when you want to.** A posture switch (`normal` / `covert` / `rogue`)
  lets a session read the room while staying invisible, or drop fully off the hive
  for private work — on every platform, because it protects you by not
  participating.
- **Honest by construction.** Coordination is pinned to same-machine transport at
  the lowest level; pointing a Pro hive command at another machine is refused with
  a clear Team/Enterprise message. Presence you see is verified for integrity, and
  stale or unverifiable sessions are shown as unknown rather than trusted.
- A **quickstart** (`docs/hivebus-quickstart.md`) walks through going from one
  session to a meshing fleet.

### Changed
- Hardened local hive safety: stale sessions read as unknown, directive delivery
  requires fresh same-session evidence, diagnostics are bounded in JSON output,
  bus access no longer doubles as directive authority, and replaying a signed
  directive under a new message is ignored.

## [0.21.0] - 2026-06-19

This release makes NeuroRouter ride out the rough patches between your agent and the model provider, so a momentary provider hiccup or network blip no longer kills your turn — while real problems still reach you honestly and immediately.

### Added
- Transient provider overloads and rate-limit surges are now smoothed over. When the provider briefly says "overloaded" or "slow down" (the kind of error that is not your actual usage limit), NeuroRouter quietly waits and retries for you instead of failing the turn, and shows you a brief operator notice that it is doing so. You see a short pause with an explanation rather than a dropped request.
- Honest handling of the failures that should not be retried. Real authentication problems, your genuine usage or quota limits, invalid requests, and policy refusals are passed straight through to you unchanged — NeuroRouter never hides a real failure behind retries.
- Immediate offline awareness. If your machine loses DNS or network connectivity, NeuroRouter tells you right away that you appear to be offline instead of silently retrying, so you are never left wondering why a request is hanging.
- Brief connection blips are absorbed. A connection that fails before your request ever reached the provider is retried on a fresh connection automatically, since nothing ran on the provider's side.
- Interrupted streaming responses are handled safely. If a streaming reply is cut off partway, NeuroRouter ends it honestly rather than risking duplicated or corrupted output — it never replays tokens you already received.

### Security
- Fixed a case where a secret written in a hash-like or reference-like shape (for example prefixed like a checksum or labeled like a fingerprint field) could be missed by the full-scrub redaction paths. Such secrets are now reliably redacted again, while routine reference values still do not raise false exposure alerts.

### Notes
- All of the above runs entirely on your machine. NeuroRouter still never calls home; the retry and offline handling are purely local reactions to responses it already receives.
- Cost figures remain estimates based on published provider pricing and may differ from your actual billed cost; token counts are measured directly.

## [0.20.0] - 2026-06-13

This release makes NeuroRouter a governed secret-handling layer between your coding agent and the model provider. Secrets that show up in tool output are caught before they reach the model, the protection fails safe rather than silently passing a secret through, and the whole behavior was checked against a large set of deliberate attack cases before shipping.

### Added
- Secrets in tool output are now caught before the model sees them. When a command your agent runs prints a token, key, or other credential, NeuroRouter replaces it with a reversible placeholder before that output enters the model's context, and shows you an operator-only notice that a secret was caught — naming the kind of secret, never the value. Plain conversation is unaffected.
- A pre-launch credential sweep. `nr launch` now does a quick, advisory scan of your workspace for credentials sitting in files and shows you a secret-free summary (how many, what kinds, which files). It never blocks your launch, never changes files, and respects your ignore rules.
- `nr doctor` now warns you when secret-bearing environment variables are being exported into your agent's environment, naming only the variable names and the risk, so you can decide whether to rotate or unset them.
- Proactive obfuscation you can pre-declare. You can now tell NeuroRouter, in config, which of your own domains, internal email addresses, and naming patterns to obfuscate on the way out — and a new `nr obfuscation-init` command suggests those entries for you from a sample log or alias file, so you don't have to remember every one by hand. Suggestions only; it never writes your config for you.
- Exposure alerts when redaction spikes. If an unusual number of secrets start flowing through a session, NeuroRouter raises a quiet advisory so you notice early. Alerts carry only counts and categories — never any secret material.
- Agent attribution is stripped from model output. "Generated with …" footers and co-author trailers that coding agents add to their output are removed before they reach you, so they never end up in your commits or pull requests. On by default, with a setting to turn it off.

### Security
- Secret protection now fails safe. If the redaction step ever errors or is interrupted, the request is held rather than forwarded — unredacted content is never passed through on a failure. This is the core guarantee behind the release, and it was verified against a large set of deliberate attack cases covering every way a secret might otherwise have slipped through.
- A protected secret is only ever restored to the legitimate destination. If model output tries to send a placeholder somewhere it shouldn't — for example an outbound command to an arbitrary address — the real secret is not substituted in; the placeholder is left unresolved.
- Coverage is now structurally complete. NeuroRouter's checks confirm that every path carrying data to the model provider routes through secret protection, so a future code path can't quietly bypass it.
- Placeholders are wire-safe across providers. The reversible placeholders NeuroRouter uses no longer break OpenAI-compatible proxies (such as LiteLLM/DashScope-style gateways), and placeholders issued by NeuroRouter can never collide with those from other tools.

### Fixed
- Fixed a request failure that could occur when obfuscating content containing certain escape sequences (backslashes, emoji, Windows-style paths). Such content now round-trips correctly.
- Long Opus sessions on the 1-million-token window now report the correct context size consistently instead of occasionally flipping back to the smaller default.
- Several provider-compatibility fixes for OpenAI-compatible and Qwen/DashScope routes, Windows reliability improvements, and more accurate cache-usage accounting.

## [0.19.10] - 2026-06-01

### Fixed
- Resuming a long Opus session now keeps your large context window automatically. Previously a resumed session could come back on the smaller default window, forcing you to re-select the model by hand each time; NeuroRouter now restores the right model on resume while leaving fresh sessions and any model you set yourself untouched.
- The output-safety check that warns when model output looks like it could carry hidden data is now much quieter on ordinary content. Normal encoded payloads, images, and routine tool output no longer trigger warnings, and explaining or quoting markup in plain text no longer trips a false alarm — so the warnings you do see are worth reading. Genuine credential-bearing or hidden-payload output is still caught, including when a response is cut off or interrupted partway through.
- Secret protection now stays consistent across agent and sub-agent turns, including when a streaming connection drops and reconnects, so a value protected on one turn is reliably restored on later turns of the same conversation and never leaks between unrelated conversations.

## [0.19.9] - 2026-05-26

### Added
- Under `nr launch claude`, NeuroRouter now owns the shape of file-read tool output end-to-end. v1 forwards file contents byte-for-byte by default and protects value-bearing strings (URLs, signed payloads, credential-shaped identifiers) so they reach the agent intact. Future per-tool shapers will ship in dedicated releases.
- New operator-only sidecar channel for messages NeuroRouter shows the human operator without exposing them to the model. Operator notices stay out of provider-bound requests and the model's context by construction.
- New `nr shape-tool` command observes which local CLI tools produce noisy output during a session, kept per-machine and opt-in. Operators see local recommendations for tools worth shaping; nothing is sent off the machine.

### Changed
- NeuroRouter Pro no longer recommends or relies on third-party command-output compactors for NR-mediated sessions. NeuroRouter owns the shape end-to-end under `nr launch`.

## [0.19.8] - 2026-05-22

### Fixed
- Improved Claude cache compatibility so stable cached request prefixes are preserved while NeuroRouter continues shaping only the live mutable portion of the request.
- Fixed a Claude launch failure where a fresh session could fail after a tool step with `Invalid signature in thinking block`.

## [0.19.7] - 2026-05-19

### Fixed
- Improved Homebrew release publishing reliability so future patch releases update the tap safely before users reinstall.

## [0.19.6] - 2026-05-19

### Fixed
- Fixed `nr context-meter --session` so compact resumed-session IDs shown in the table can be used as filters, and fixed session-filtered `--json` output in the default output mode.

## [0.19.5] - 2026-05-19

### Fixed
- Improved `nr context-meter` for Claude resume sessions so token fields are populated when the provider returns usage, and missing provider usage is reported more clearly.

## [0.19.4] - 2026-05-19

### Fixed
- Improved `nr context-meter --last` output so rows from different sessions are visibly distinct, and fixed JSON timestamps that could show as year 0001.

## [0.19.3] - 2026-05-19

### Fixed
- Fixed `nr context-meter --json` and `nr context-meter --explain ... --json` in the default output mode so support-safe measurement fields are shown instead of being hidden.

## [0.19.2] - 2026-05-18

### Added
- Added `nr context-meter`, a support-safe diagnostic command for investigating context-window changes, resume boundaries, cache effects, and model denominator changes without storing prompts, responses, or raw provider payloads.

### Fixed
- Improved context-meter coverage and reporting so passthrough traffic, translated chat responses, cached-token accounting, retention cleanup, signed deltas, and explain output stay consistent and support-safe.

## [0.19.1] - 2026-05-18

### Fixed
- Made `nr launch codex` stop before opening Codex when no `OPENAI_API_KEY` is configured, with a clear API-key setup path instead of starting a session that cannot write through the provider.

## [0.19.0] - 2026-05-17

### Added
- Added governed run foundations for future team workflows, including deterministic run shaping, tool-call decisions, audit evidence, and tier-aware safety checks.
- Added `nr approve <run-id> <call-id>` so a supervised run can pause for an operator decision and continue without storing raw tool payloads.

## [0.18.83] - 2026-05-16

### Fixed
- Improved `nr launch claude --resume` reliability for long-running Claude Code sessions that could start successfully and later fail with a provider resume error.

## [0.18.82] - 2026-05-16

### Fixed
- Improved customer-safe launcher logs so environment variable names stay readable and repeated words are cleaned up in startup warnings.

## [0.18.81] - 2026-05-16

### Fixed
- Improved `nr launch claude --resume` compatibility with account-auth Claude Code sessions by preserving native resume traffic through startup.

## [0.18.80] - 2026-05-16

### Fixed
- Improved Claude Code resume compatibility so `nr launch claude --resume` can repair bounded local-session metadata issues and warn instead of blocking older mixed-auth sessions before Claude can try to resume.
- Improved `nr launch` error output so child-client resume failures no longer look like launcher command misuse.
- Improved `nr launch` terminal cleanup so exiting Codex or Claude Code is less likely to leave escape text in the shell prompt.

## [0.18.79] - 2026-05-16

### Fixed
- Added a launch supervision check that warns if a started Codex or Claude Code session exits without sending any requests through NeuroRouter.

## [0.18.78] - 2026-05-16

### Fixed
- Improved `nr launch` reliability for Codex and Claude Code by avoiding stale local proxy reuse, stopping launcher-owned proxies after the client exits, and cleaning up terminal input without showing cleanup errors.

## [0.18.77] - 2026-05-16

### Fixed
- Improved `nr launch` cleanup so exiting Codex or other interactive clients restores the shell terminal state reliably.

## [0.18.76] - 2026-05-16

### Added
- Added safer Claude Code resume handling. `nr launch claude --resume`, `nr launch claude --continue`, and `nr resume <session>` now inspect local resume state before Claude opens, recover bounded local-session issues automatically, and stop with a clear API-key recovery command when an older long-running session should not be resumed through account passthrough.

## [0.18.75] - 2026-05-15

### Fixed
- Improved `nr launch claude` resume startup by repairing a known malformed Claude Code local-session metadata shape before Claude opens the resumed conversation.

## [0.18.74] - 2026-05-15

### Fixed
- Fixed Claude Code resume and interactive picker behavior when launched through `nr launch claude`.
- Allowed `nr config set protect-policy redact` as an alias for the existing `protect_policy` setting.

## [0.18.73] - 2026-05-15

### Fixed
- Cleaned default help text for support and task-smoke commands so first-run output stays customer-safe.

## [0.18.72] - 2026-05-15

### Added
- Added OAuth-first `nr launch claude` and `nr launch codex` wrappers so Claude Code and Codex can start through a supervised local proxy with less manual setup.
- Added short launcher commands for common clients, including Claude Code, Codex, Cursor, and Aider.

### Improved
- Made default command output and launcher logs customer-safe while keeping diagnostic detail available through explicit operator modes.
- Reduced launcher terminal noise by moving proxy diagnostics into a local log file.

### Fixed
- Fixed public launcher startup in standard builds.
- Fixed the support capture test build path.

## [0.18.71] - 2026-05-14

### Added
- Added `nr` as a short command name for NeuroRouter Pro. The long `neurorouter` command remains supported.
- Added clearer rocket execution evidence for transport checks, cost review, and structured failure reports.

### Improved
- Improved rocket startup and work-order alignment checks so focused execution fails closed when required control-plane state is unavailable.
- Improved model pricing and provider capability evidence used by routing and cost reporting.

## [0.18.70] - 2026-04-27

### Changed
- Public release verification now explicitly confirms customer builds reject diagnostic-only modes while keeping evidence and audit commands available.
- Clarified the Codex setup path with a Responses provider profile, resume commands, and explicit model-selection examples.

### Fixed
- Improved Claude session reliability while reducing completed tool workflow overhead.
- Corrected Claude auth routing so stable Claude traffic uses Anthropic API-key auth by default.
- Restored explicit local diagnostic support for client/account auth passthrough without changing public or hosted safety defaults.
- Improved Codex resume stability when an old workspace marker disagrees with the active session.
- Improved long Codex session shaping so important continuity evidence is preserved or recovery is triggered before forwarding.

## [0.18.68] - 2026-04-26

### Added
- Added the Context Compiler for preserving task intent while reducing long-session context size.
- Added customer-safe evidence reports and support bundles for troubleshooting without raw prompt bodies.
- Added Codex rescue tooling for stuck or overgrown sessions.

### Changed
- Public Brew, Scoop, and GitHub release builds now expose evidence and support-safe views instead of raw engineering captures.
- Improved long-session continuity behavior for Claude and Codex workflows.

### Fixed
- Improved recovery behavior for stale or repeated agent loops.
- Improved provider cooldown handling and auth guidance.
- Improved clean-session recovery behavior during long-running work.
