# IronClaw Documentation

> Comprehensive developer reference for [IronClaw](https://github.com/nearai/ironclaw) v0.25.0
> — a secure, self-hosted personal AI assistant written in Rust.

**Documentation set for IronClaw v0.25.0, validated against release tag `v0.25.0` (2026-04-11).**

---

## Contents

| Document | Lines | Description |
|----------|------:|-------------|
| [CHANGELOG.md](CHANGELOG.md) | ~120 | Mirrored release changelog for the latest released versions in scope |
| [INSTALLATION.md](INSTALLATION.md) | ~715 | Installation, configuration, service setup, troubleshooting |
| [LLM_PROVIDERS.md](LLM_PROVIDERS.md) | ~178 | LLM backend configuration quick guide (NEAR AI, OpenAI, Anthropic, Ollama, OpenAI-compatible) |
| [TELEGRAM_SETUP.md](TELEGRAM_SETUP.md) | ~137 | Telegram channel setup with DM pairing flow and webhook/polling modes |
| [SIGNAL_SETUP.md](SIGNAL_SETUP.md) | ~200 | Signal channel setup via signal-cli HTTP daemon |
| [BUILDING_CHANNELS.md](BUILDING_CHANNELS.md) | ~442 | WASM channel authoring and build/deploy workflow |
| [NETWORK_SECURITY.md](NETWORK_SECURITY.md) | ~190 | Network-facing surfaces, auth boundaries, egress controls, deployment caveats |
| [smart-routing-spec.md](smart-routing-spec.md) | ~180 | Smart routing design spec and tiered model-selection rationale |
| [ARCHITECTURE.md](ARCHITECTURE.md) | ~877 | Master architecture: modules, data flows, diagrams |
| [AGENT_README.md](AGENT_README.md) | ~1245 | Agent reference: errors, config, code review patterns |
| [analysis/agent.md](analysis/agent.md) | ~930 | Agent loop, sessions, jobs, routines, heartbeat, cost guard |
| [analysis/channels.md](analysis/channels.md) | ~1017 | REPL, web gateway, HTTP, WASM, webhook channels + full API routes |
| [analysis/cli.md](analysis/cli.md) | ~505 | CLI subcommands, doctor, service manager, MCP, registry |
| [analysis/config.md](analysis/config.md) | ~1034 | Configuration system — exhaustive env var reference |
| [analysis/llm.md](analysis/llm.md) | ~803 | LLM backends, multi-provider, retry, cost guard, schema fix |
| [analysis/safety-sandbox.md](analysis/safety-sandbox.md) | ~520 | Safety layer, WASM sandbox, Docker orchestrator, SSRF proxy |
| [analysis/skills-extensions.md](analysis/skills-extensions.md) | ~736 | Skills system, WASM channels, extensions, hooks |
| [analysis/tools.md](analysis/tools.md) | ~1515 | Tool system, all built-in tools, MCP client, WASM tools, builder |
| [analysis/tunnels-pairing.md](analysis/tunnels-pairing.md) | ~347 | Tunnels (cloudflare/ngrok/tailscale/custom), mobile pairing |
| [analysis/worker-orchestrator.md](analysis/worker-orchestrator.md) | ~485 | Worker runtime, Claude bridge, proxy LLM, Docker sandbox |
| [analysis/workspace-memory.md](analysis/workspace-memory.md) | ~730 | Workspace FS, semantic memory, embeddings, hybrid search |
| [analysis/secrets-keychain.md](analysis/secrets-keychain.md) | ~349 | Secrets store, keychain, AES-GCM crypto, credential injection |

---

## About IronClaw

IronClaw is a Rust-based personal AI assistant built by [NEAR AI](https://near.ai) with:

- **Multi-channel**: REPL, web gateway (axum), HTTP webhooks, WASM plugin channels (incl. Feishu/Lark), native Signal channel
- **Security-first**: WASM sandbox (wasmtime), Docker isolation (bollard), credential injection, SSRF proxy, multi-tenant isolation
- **Graduated command approval**: Low/Medium/High risk levels with per-tool reasoning threaded through all surfaces
- **Self-expanding**: Dynamic WASM tool builder, MCP protocol client, plugin architecture
- **Persistent memory**: Hybrid FTS+vector search (RRF), workspace filesystem, identity files, layered memory with privacy redirect
- **Multiple LLM backends**: NEAR AI, Anthropic, OpenAI, Ollama, OpenAI-compatible, Tinfoil, MiniMax, Z.AI, Codex/ChatGPT, GitHub Copilot, Gemini (Cloud Code API OAuth), OpenAI Codex (ChatGPT subscription)
- **Dual database**: libSQL (embedded, no server required) or PostgreSQL (with pgvector)

### Source Module Statistics (v0.25.0)

| Module | Files | Description |
|--------|------:|-------------|
| `tools/` | 69 | Tool system: built-in tools, MCP, WASM sandbox, dynamic tool builder, rate limiter, HTML-to-Markdown |
| `channels/` | 64 | Channels: REPL, web gateway, HTTP, Signal, Telegram, Discord, Slack, other WASM plugins |
| `agent/` | 22 | Agent runtime: loop, sessions, jobs, routines, heartbeat, context compaction |
| `config/` | 24 | Configuration: env var loading and all config structs |
| `workspace/` | 14 | Memory, embeddings, hybrid FTS+vector search, privacy controls |
| `llm/` | 35 | LLM backends, retries, circuit breaking, tool routing, cost tracking |
| `tunnel/` | 6 | Tunnels: cloudflare, ngrok, tailscale, custom |
| `secrets/` | 5 | Keychain, AES-256-GCM crypto, credential injection (OsRng throughout) |
| `safety/` | 7 | Prompt injection defense, leak detection, secret scanning, policy enforcement (`crates/ironclaw_safety/src`) |
| `worker/` | 8 | Docker worker: runtime, LLM bridge, proxy |
| **Total (`src/`)** | **394** | All files under `src/` at `v0.25.0` (including 394 Rust source files) |

---

## Quick Start (macOS, local mode)

**Fastest option — Homebrew:**
```bash
brew install ironclaw
```

**Or install pre-built binary:**
```bash
curl --proto '=https' --tlsv1.2 -LsSf \
  https://github.com/nearai/ironclaw/releases/latest/download/ironclaw-installer.sh | sh
```

**Or build from source (libSQL, no PostgreSQL required):**
```bash
git clone https://github.com/nearai/ironclaw ~/src/ironclaw
cd ~/src/ironclaw
cargo build --release --no-default-features --features libsql
install -m 755 target/release/ironclaw ~/.local/bin/ironclaw
```

**Configure and run:**
```bash
mkdir -p ~/.ironclaw
cat > ~/.ironclaw/.env <<'EOF'
DATABASE_BACKEND=libsql
LLM_BACKEND=openai
OPENAI_API_KEY=sk-proj-...
GATEWAY_ENABLED=true
GATEWAY_PORT=3000
GATEWAY_AUTH_TOKEN=REPLACE_WITH_SECURE_TOKEN
CLI_ENABLED=false
RUST_LOG=ironclaw=info
EOF

# Generate a secure token: openssl rand -hex 32
# Replace REPLACE_WITH_SECURE_TOKEN above with the output

# Run
ironclaw --no-onboard

# Test
curl http://127.0.0.1:3000/api/health
```

See [INSTALLATION.md](INSTALLATION.md) for complete setup and deployment, [LLM_PROVIDERS.md](LLM_PROVIDERS.md) for backend-specific examples, [TELEGRAM_SETUP.md](TELEGRAM_SETUP.md) or [SIGNAL_SETUP.md](SIGNAL_SETUP.md) for messaging channel setup, [BUILDING_CHANNELS.md](BUILDING_CHANNELS.md) for custom WASM channels, [NETWORK_SECURITY.md](NETWORK_SECURITY.md) for network threat-surface review, and [smart-routing-spec.md](smart-routing-spec.md) for the released smart-routing design.

---

## What's New

### v0.25.0 (2026-04-11)

### Added

- *(tools)* production-grade coding tools, file history, and skills ([#2025](https://github.com/nearai/ironclaw/pull/2025))
- add extensible deployment profiles (IRONCLAW_PROFILE) ([#2203](https://github.com/nearai/ironclaw/pull/2203))
- *(gateway)* extract gateway frontend into `ironclaw_gateway` crate with widget system ([#1725](https://github.com/nearai/ironclaw/pull/1725))
- *(railway)* build staging target with pre-bundled WASM extensions ([#2219](https://github.com/nearai/ironclaw/pull/2219))
- *(docker)* pre-bundle WASM extensions in staging image ([#2210](https://github.com/nearai/ironclaw/pull/2210))
- *(tui)* ship TUI in default binary ([#2195](https://github.com/nearai/ironclaw/pull/2195))
- *(admin)* admin tool policy to disable tools for users ([#2154](https://github.com/nearai/ironclaw/pull/2154))
- *(web)* add scroll-to-bottom arrow in gateway chat ([#2202](https://github.com/nearai/ironclaw/pull/2202))
- *(skills)* commitments system for personal assistant workflow persistence ([#1736](https://github.com/nearai/ironclaw/pull/1736))

### Fixed

- *(ci)* bump 5 channel versions + fix lifetime desync in panics check ([#2300](https://github.com/nearai/ironclaw/pull/2300))
- *(test)* case-insensitive hint matching in TraceLlm step_matches ([#2292](https://github.com/nearai/ironclaw/pull/2292))
- *(v2)* tool naming, auth gates, schema flatten, WASM traps, workspace race ([#2209](https://github.com/nearai/ironclaw/pull/2209))
- *(ci)* resolve 4 staging test failures ([#2273](https://github.com/nearai/ironclaw/pull/2273))
- *(oauth)* use localhost for redirect URI when bound to `0.0.0.0` ([#2247](https://github.com/nearai/ironclaw/pull/2247))
- *(bridge)* sanitize auth_url on engine v2 path ([#2206](https://github.com/nearai/ironclaw/pull/2206)) ([#2215](https://github.com/nearai/ironclaw/pull/2215))
- *(safety)* add credential patterns and sensitive path blocklist ([#1675](https://github.com/nearai/ironclaw/pull/1675))
- *(channels)* allow telegram WASM channel names ([#2051](https://github.com/nearai/ironclaw/pull/2051))
- *(ownership)* remove silent cross-tenant credential fallback ([#2099](https://github.com/nearai/ironclaw/pull/2099))
- *(web)* emit Done after response for SSE ordering ([#2079](https://github.com/nearai/ironclaw/pull/2079)) ([#2104](https://github.com/nearai/ironclaw/pull/2104))
- *(worker)* improve command execution validation ([#1692](https://github.com/nearai/ironclaw/pull/1692))
- *(llm)* invert reasoning default — unknown models skip `think/final` tags ([#1952](https://github.com/nearai/ironclaw/pull/1952))

### Other

- add amazon tutorial ([#2261](https://github.com/nearai/ironclaw/pull/2261))
- [codex] Stabilize auth readiness and gate flows ([#2050](https://github.com/nearai/ironclaw/pull/2050))
- Add mintlify docs ([#2189](https://github.com/nearai/ironclaw/pull/2189))
- *(ci)* add Dependabot and pin GitHub Actions by SHA ([#2043](https://github.com/nearai/ironclaw/pull/2043))
- Build and staging reliability and release workflow updates

### v0.24.0 (2026-03-31)

### Added

- *(gateway)* OIDC JWT authentication for reverse-proxy deployments ([#1463](https://github.com/nearai/ironclaw/pull/1463))
- support custom LLM provider configuration via web UI ([#1340](https://github.com/nearai/ironclaw/pull/1340))
- *(skills)* recursive bundle directory scanning for skill discovery ([#1667](https://github.com/nearai/ironclaw/pull/1667))
- *(discord)* add gateway channel flow in WASM ([#944](https://github.com/nearai/ironclaw/pull/944))
- *(gateway)* add OpenAI Responses API endpoints ([#1656](https://github.com/nearai/ironclaw/pull/1656))

### Fixed

- *(routines)* clone `Arc` before await in web handler event cache refresh ([#1756](https://github.com/nearai/ironclaw/pull/1756))
- *(slack)* respond to thread replies without requiring `@mention` ([#1405](https://github.com/nearai/ironclaw/pull/1405))
- *(auth)* make shared Google tool status scope-aware ([#1532](https://github.com/nearai/ironclaw/pull/1532))
- *(web)* redact database error details from API responses ([#1711](https://github.com/nearai/ironclaw/pull/1711))

### Other

- Stabilize MCP refresh regression tests ([#1772](https://github.com/nearai/ironclaw/pull/1772))
- Fix hosted MCP OAuth refresh flow ([#1767](https://github.com/nearai/ironclaw/pull/1767))
- Track routine verification state across updates ([#1716](https://github.com/nearai/ironclaw/pull/1716))

### v0.23.0 (2026-03-27)

### Added

- **Multi-tenant isolation phases 2-4** — complete per-user workspace isolation with multi-tenant auth ([#1614](https://github.com/nearai/ironclaw/pull/1614))
- **Direct hosted OAuth callbacks** with proxy auth token ([#1684](https://github.com/nearai/ironclaw/pull/1684))

### Fixed

- **MCP Streamable HTTP** — handle 202 Accepted responses and wire session manager for Streamable HTTP transport ([#1437](https://github.com/nearai/ironclaw/pull/1437))
- **Truncated tool call handling** — discard truncated tool calls when `finish_reason == Length` instead of passing malformed JSON to tools ([#1631](https://github.com/nearai/ironclaw/pull/1631), [#1632](https://github.com/nearai/ironclaw/pull/1632))
- **Routine recovery** — recover delete name after failed update fallback ([#1108](https://github.com/nearai/ironclaw/pull/1108))
- Channel-relay auth dead-end, observability, and URL override ([#1681](https://github.com/nearai/ironclaw/pull/1681))
- XML tool-call recovery filtered by context ([#1641](https://github.com/nearai/ironclaw/pull/1641))

---

### v0.22.0 (2026-03-25)

### Added

- **Per-tool reasoning** — thread reasoning through provider, session, and all surfaces so the agent explains _why_ it chose each tool ([#1513](https://github.com/nearai/ironclaw/pull/1513))
- **`ironclaw models` subcommands** — `list`, `status`, `set`, `set-provider` for managing LLM model configuration from the CLI ([#1043](https://github.com/nearai/ironclaw/pull/1043))
- **`ironclaw hooks list`** — new subcommand to inspect registered lifecycle hooks ([#1023](https://github.com/nearai/ironclaw/pull/1023))
- **Multi-tenant auth** with per-user workspace isolation ([#1118](https://github.com/nearai/ironclaw/pull/1118))
- **Gemini OAuth** — full Gemini CLI OAuth integration with Google Cloud Code API ([#1356](https://github.com/nearai/ironclaw/pull/1356))
- **GitHub Copilot provider** — use GitHub Copilot as an LLM backend ([#1512](https://github.com/nearai/ironclaw/pull/1512))
- **OpenAI Codex provider** — use OpenAI Codex via ChatGPT subscription ([#1461](https://github.com/nearai/ironclaw/pull/1461))
- **Graduated command approval** — Low/Medium/High risk levels for shell commands (closes #172) ([#368](https://github.com/nearai/ironclaw/pull/368))
- **Message queuing** — queue and merge messages during active turns instead of dropping them ([#1412](https://github.com/nearai/ironclaw/pull/1412))
- **Layered memory** with sensitivity-based privacy redirect ([#1112](https://github.com/nearai/ironclaw/pull/1112))
- **Webhook trigger endpoint** — public endpoint for triggering routines externally ([#736](https://github.com/nearai/ironclaw/pull/736))
- **Light/dark/system theme** toggle for the web gateway ([#1457](https://github.com/nearai/ironclaw/pull/1457))
- **UX overhaul** — complete design system, onboarding, and web polish ([#1277](https://github.com/nearai/ironclaw/pull/1277))
- **`stuck_threshold` activation** — time-based stuck job detection ([#1234](https://github.com/nearai/ironclaw/pull/1234))
- **Chat onboarding** and routine advisor ([#927](https://github.com/nearai/ironclaw/pull/927))

### Fixed (Security)

- Validate embedding base URLs to prevent SSRF ([#1221](https://github.com/nearai/ironclaw/pull/1221))
- Escape tool output XML content and remove misleading sanitized attr ([#1067](https://github.com/nearai/ironclaw/pull/1067))
- Reject malformed `ic2.*` states in hosted OAuth decoder ([#1441](https://github.com/nearai/ironclaw/pull/1441), [#1454](https://github.com/nearai/ironclaw/pull/1454))
- Patch rustls-webpki vulnerability (RUSTSEC-2026-0049)
- Post-merge review sweep — 8 fixes across security, perf, and correctness ([#1550](https://github.com/nearai/ironclaw/pull/1550))

---

### v0.21.0 (2026-03-20)

### Added

- **Structured fallback deliverables** — failed or stuck jobs now produce structured partial deliverables instead of bare error strings ([#236](https://github.com/nearai/ironclaw/pull/236))
- **LRU embedding cache** — workspace search avoids redundant embedding calls via an in-memory LRU cache ([#1423](https://github.com/nearai/ironclaw/pull/1423))
- **Webhook callbacks for relay events** — receive channel-relay events via configurable HTTP callbacks ([#1254](https://github.com/nearai/ironclaw/pull/1254))

### Other

- **Generic hosted OAuth and MCP auth** — auth flows generalized beyond Anthropic/OpenAI to support arbitrary OAuth providers ([#1375](https://github.com/nearai/ironclaw/pull/1375))
- Auto-approve fix for credentialed HTTP requests ([#1257](https://github.com/nearai/ironclaw/pull/1257))

---

### v0.20.0 (2026-03-19)

### Added

- **Self-repair `stuck_threshold` wire** — stuck_threshold, its store, and the builder are now fully wired so the agent self-repairs stuck jobs ([#712](https://github.com/nearai/ironclaw/pull/712))
- **FaultInjector testing framework** — inject failures into `StubLlm` for resilience testing ([#1233](https://github.com/nearai/ironclaw/pull/1233))
- **Unified settings page** — single settings page with subtabs in the web gateway ([#1191](https://github.com/nearai/ironclaw/pull/1191))
- **MiniMax M2.7 upgrade** — default MiniMax model upgraded to M2.7 ([#1357](https://github.com/nearai/ironclaw/pull/1357))

### Fixed

- Routine concurrency: `full_job` runs stay running until linked job completion ([#1374](https://github.com/nearai/ironclaw/pull/1374), [#1372](https://github.com/nearai/ironclaw/pull/1372))
- MCP: retry after missing session id errors ([#1355](https://github.com/nearai/ironclaw/pull/1355))
- Telegram: preserve polling after secret-blocked updates ([#1353](https://github.com/nearai/ironclaw/pull/1353))
- LLM: cap retry-after delays ([#1351](https://github.com/nearai/ironclaw/pull/1351))
- Rate limiter returns `retry_after: None` instead of a duration ([#1269](https://github.com/nearai/ironclaw/pull/1269))
- Removed debug_assert guards that panic on valid error paths ([#1385](https://github.com/nearai/ironclaw/pull/1385))

---

### v0.19.0 (2026-03-17)

### Added

- **Feishu/Lark WASM channel** — Talk to your agent through a Feishu or Lark bot ([#1110](https://github.com/nearai/ironclaw/pull/1110))
- **New LLM providers**: MiniMax (built-in, `LLM_BACKEND=minimax`), Z.AI/GLM-5 (built-in, `LLM_BACKEND=zai`), Codex/ChatGPT via `LLM_USE_CODEX_AUTH=true` ([#940](https://github.com/nearai/ironclaw/pull/940), [#938](https://github.com/nearai/ironclaw/pull/938), [#693](https://github.com/nearai/ironclaw/pull/693))
- **`ironclaw logs`** — Gateway log access CLI: live follow (`-f`), JSON output, log-level control ([#1105](https://github.com/nearai/ironclaw/pull/1105))
- **`ironclaw cron`** — Alias for `ironclaw routines` for intuitive cron scheduling ([#1017](https://github.com/nearai/ironclaw/pull/1017))
- **`ironclaw channels list`** and **`ironclaw skills list/search/info`** — new CLI discovery commands ([#933](https://github.com/nearai/ironclaw/pull/933), [#918](https://github.com/nearai/ironclaw/pull/918))
- **Heartbeat `fire_at`** — Time-of-day scheduling with IANA timezone support ([#1029](https://github.com/nearai/ironclaw/pull/1029))
- **i18n support** — Chinese (zh-CN) and English UI translations ([#929](https://github.com/nearai/ironclaw/pull/929))
- Slack approval buttons for tool execution in DMs ([#796](https://github.com/nearai/ironclaw/pull/796))
- Context-llm tool support ([#616](https://github.com/nearai/ironclaw/pull/616))
- Configurable hybrid search fusion strategy ([#234](https://github.com/nearai/ironclaw/pull/234))
- Import OpenClaw memory, history, and settings ([#903](https://github.com/nearai/ironclaw/pull/903))
- ASCII art banner during onboarding ([#851](https://github.com/nearai/ironclaw/pull/851))
- Sandbox retry logic for transient container failures ([#1232](https://github.com/nearai/ironclaw/pull/1232))

### Fixed (Security)

- Webhook auth migrated to HMAC-SHA256 `X-Webhook-Signature` header (was in body) ([#970](https://github.com/nearai/ironclaw/pull/970))
- Webhook server defaults to loopback when tunnel is configured ([#1194](https://github.com/nearai/ironclaw/pull/1194))
- Content-Security-Policy header added to web gateway ([#966](https://github.com/nearai/ironclaw/pull/966))
- Metadata spoofing prevention for internal job monitor flag ([#1195](https://github.com/nearai/ironclaw/pull/1195))
- Anthropic OAuth token persistence after Keychain re-read ([#1213](https://github.com/nearai/ironclaw/pull/1213))
- MCP: handle 400 auth errors, clear auth mode after OAuth, trim tokens ([#1158](https://github.com/nearai/ironclaw/pull/1158))
- Eliminated panic paths in production code ([#1184](https://github.com/nearai/ironclaw/pull/1184))
- Telegram owner verification during hot activation ([#1157](https://github.com/nearai/ironclaw/pull/1157))

### v0.18.0 (2026-03-11)

### Other

- promote staging to main from v0.17.0 lineage
- update WASM artifact SHA256 checksums (release maintenance)

### v0.17.0 (2026-03-10)

### Added

- add AWS Bedrock LLM provider via native Converse API
- support more providers in the declarative registry (Gemini, io.net, Mistral, Yandex, Cloudflare)
- MCP transport abstraction with stdio and UDS support and OAuth fixes
- background sandbox reaper for orphaned Docker containers
- add support for image messages across all channels
- route `routines` approvals and message metadata from per-job context

### Fixed

- deterministic behavior in smart routing and tool-call handling regressions
- stable token expiry and security token handling in network tooling
- workflow and CI reliability around staging promotions
- preserve tool-call history and `tool_result` serialization behavior

### v0.16.1 (2026-03-06)

#### Fixed

- revert WASM artifact SHA256 checksums to null ([#627](https://github.com/nearai/ironclaw/pull/627))

---

### v0.16.0 (2026-03-06)

#### Added

- **Smart routing redesign**: 13-dimension complexity scorer with four tiers (Flash/Standard/Pro/Frontier), pattern overrides, and cascade mode ([#529](https://github.com/nearai/ironclaw/pull/529))
- **WASM extension versioning** with WIT compatibility checks — `wit_version` field in capabilities files, DB migration `V10__wasm_versioning.sql` ([#592](https://github.com/nearai/ironclaw/pull/592))
- **HMAC-SHA256 webhook signature validation for Slack** — 5-minute replay window, constant-time comparison ([#588](https://github.com/nearai/ironclaw/pull/588))
- **Graceful restart** via `/restart` command (web-only, Docker-only) — `RestartTool` triggers `std::process::exit(0)` via entrypoint loop ([#531](https://github.com/nearai/ironclaw/pull/531))
- **Unified `http` tool** — merges `web_fetch` into `http`; conditional approval (plain GET = never, auth/POST = always); `IRONCLAW_IN_DOCKER` env var ([#578](https://github.com/nearai/ironclaw/pull/578))
- *(e2e)* extensions tab tests, CI parallelization, and 3 production bug fixes ([#584](https://github.com/nearai/ironclaw/pull/584))

#### Fixed

- *(llm)* fix reasoning model response parsing bugs ([#580](https://github.com/nearai/ironclaw/pull/580))
- Telegram channel accepts group messages from all users if `respond_to_all_group_messages=true` ([#590](https://github.com/nearai/ironclaw/pull/590))
- *(security)* use `OsRng` for all security-critical key and token generation ([#519](https://github.com/nearai/ironclaw/pull/519))
- prevent concurrent memory hygiene passes and Windows file lock errors ([#535](https://github.com/nearai/ironclaw/pull/535))
- sort `tool_definitions()` for deterministic LLM tool ordering ([#582](https://github.com/nearai/ironclaw/pull/582))

#### Other

- *(llm)* complete response cache — `set_model` invalidation, stats logging, sync mutex ([#290](https://github.com/nearai/ironclaw/pull/290))

---

### v0.15.0 (2026-03-04)

#### Added

- *(oauth)* route callbacks through web gateway for hosted instances ([#555](https://github.com/nearai/ironclaw/pull/555))
- *(web)* show error details for failed tool calls ([#490](https://github.com/nearai/ironclaw/pull/490))
- *(extensions)* improve auth UX and add load-time validation ([#536](https://github.com/nearai/ironclaw/pull/536))
- add local-test skill and Dockerfile.test for web gateway testing ([#524](https://github.com/nearai/ironclaw/pull/524))

#### Fixed

- *(security)* restrict query-token auth to SSE endpoints only ([#528](https://github.com/nearai/ironclaw/pull/528))
- *(ci)* flush profraw coverage data in E2E teardown ([#550](https://github.com/nearai/ironclaw/pull/550))
- *(wasm)* coerce string parameters to schema-declared types ([#498](https://github.com/nearai/ironclaw/pull/498))
- *(agent)* strip leaked [Called tool ...] text from responses ([#497](https://github.com/nearai/ironclaw/pull/497))
- *(web)* reset job list UI on restart failure ([#499](https://github.com/nearai/ironclaw/pull/499))
- *(security)* replace .unwrap() panics in pairing store with proper error handling ([#515](https://github.com/nearai/ironclaw/pull/515))

#### Other

- Fix UTF-8 unsafe truncation in sandbox log capture ([#359](https://github.com/nearai/ironclaw/pull/359))
- enhance coverage with feature matrix, postgres, and E2E ([#523](https://github.com/nearai/ironclaw/pull/523))

### v0.14.0 (2026-03-04)

#### Added

- add OAuth support for WASM tools in web gateway ([#489](https://github.com/nearai/ironclaw/pull/489))
- *(web)* jobs UI parity for non-sandbox mode ([#491](https://github.com/nearai/ironclaw/pull/491))
- *(workspace)* add TOOLS.md, BOOTSTRAP.md, and disk-to-DB import ([#477](https://github.com/nearai/ironclaw/pull/477))

#### Removed

- Okta WASM tool ([#506](https://github.com/nearai/ironclaw/pull/506))

#### Fixed

- *(web)* mobile browser bar obscures chat input ([#508](https://github.com/nearai/ironclaw/pull/508))
- *(web)* assign unique thread_id to manual routine triggers ([#500](https://github.com/nearai/ironclaw/pull/500))
- *(web)* refresh routine UI after Run Now trigger ([#501](https://github.com/nearai/ironclaw/pull/501))
- *(skills)* use slug for skill download URL from ClawHub ([#502](https://github.com/nearai/ironclaw/pull/502))
- *(workspace)* thread document path through search results ([#503](https://github.com/nearai/ironclaw/pull/503))
- *(workspace)* import custom templates before seeding defaults ([#505](https://github.com/nearai/ironclaw/pull/505))
- use std::sync::RwLock in MessageTool to avoid runtime panic ([#411](https://github.com/nearai/ironclaw/pull/411))
- wire secrets store into all WASM runtime activation paths ([#479](https://github.com/nearai/ironclaw/pull/479))

#### Other

- enforce regression tests for fix commits ([#517](https://github.com/nearai/ironclaw/pull/517))
- Remove restart infrastructure, generalize WASM channel setup ([#493](https://github.com/nearai/ironclaw/pull/493))
- add code coverage with cargo-llvm-cov and Codecov ([#511](https://github.com/nearai/ironclaw/pull/511))

### v0.13.1 (2026-03-02)

#### Added

- add Brave Web Search WASM tool ([#474](https://github.com/nearai/ironclaw/pull/474))

#### Fixed

- *(web)* auto-scroll and Enter key completion for slash command autocomplete ([#475](https://github.com/nearai/ironclaw/pull/475))
- correct download URLs for telegram-mtproto and slack-tool extensions ([#470](https://github.com/nearai/ironclaw/pull/470))

### v0.13.0 (2026-03-02)

#### Added

- add tool setup command + GitHub setup schema ([#438](https://github.com/nearai/ironclaw/pull/438))
- add web_fetch built-in tool ([#435](https://github.com/nearai/ironclaw/pull/435))
- DB-backed Jobs tab + scheduler-dispatched local jobs ([#436](https://github.com/nearai/ironclaw/pull/436))
- OAuth setup UI for WASM tools + display name labels ([#437](https://github.com/nearai/ironclaw/pull/437))
- auto-detect libsql when `ironclaw.db` exists ([#399](https://github.com/nearai/ironclaw/pull/399))
- slash command autocomplete + `/status` and `/list` ([#404](https://github.com/nearai/ironclaw/pull/404))
- deliver notifications to all installed channels ([#398](https://github.com/nearai/ironclaw/pull/398))
- persist tool calls, restore approvals on thread switch, and Web UI fixes ([#382](https://github.com/nearai/ironclaw/pull/382))
- add `IRONCLAW_BASE_DIR` env var with LazyLock caching ([#397](https://github.com/nearai/ironclaw/pull/397))
- feat(signal) attachment upload and message tool ([#375](https://github.com/nearai/ironclaw/pull/375))

#### Fixed

- host-based credential injection to the WASM channel wrapper ([#421](https://github.com/nearai/ironclaw/pull/421))
- pre-validate Cloudflare tunnel token by spawning `cloudflared` ([#446](https://github.com/nearai/ironclaw/pull/446))
- quick fixes: `#330`, `#338`, `#344`, `#358`, `#417`, `#419` (bundled as [#428](https://github.com/nearai/ironclaw/pull/428))
- persist channel activation state across restarts ([#432](https://github.com/nearai/ironclaw/pull/432))
- init WASM runtime eagerly regardless of tools directory existence ([#401](https://github.com/nearai/ironclaw/pull/401))
- add TLS support for PostgreSQL connections ([#363](https://github.com/nearai/ironclaw/pull/363), [#427](https://github.com/nearai/ironclaw/pull/427))
- scan inbound messages for leaked secrets ([#433](https://github.com/nearai/ironclaw/pull/433))
- use `tailscale funnel --bg` for proper tunnel setup ([#430](https://github.com/nearai/ironclaw/pull/430))
- normalize secret names to lowercase for case-insensitive matching ([#413](https://github.com/nearai/ironclaw/pull/413), [#431](https://github.com/nearai/ironclaw/pull/431))
- persist model name to `.env` so dotted names survive restart ([#426](https://github.com/nearai/ironclaw/pull/426))
- setup flow validates cloudflared binary and tunnel token ([#424](https://github.com/nearai/ironclaw/pull/424))
- setup flow validates PostgreSQL version and pgvector availability before migrations ([#423](https://github.com/nearai/ironclaw/pull/423))
- guard `zsh compdef` call to prevent pre-compinit errors ([#422](https://github.com/nearai/ironclaw/pull/422))
- Telegram: remove restart button and validate token on setup ([#434](https://github.com/nearai/ironclaw/pull/434))
- web UI routines tab shows all routines regardless of creating channel ([#391](https://github.com/nearai/ironclaw/pull/391))
- Discord Ed25519 signature verification and capabilities header alias fixes ([#148](https://github.com/nearai/ironclaw/pull/148), [#372](https://github.com/nearai/ironclaw/pull/372))
- prevent duplicate WASM channel activation on startup ([#390](https://github.com/nearai/ironclaw/pull/390))

#### Other

- rename `WasmBuildable::repo_url` to `source_dir` ([#445](https://github.com/nearai/ironclaw/pull/445))
- improve `--help` with `about`, `examples`, and `color` output ([#371](https://github.com/nearai/ironclaw/pull/371))
- add automated QA: schema validator, CI matrix, Docker build, and `P1` test coverage ([#353](https://github.com/nearai/ironclaw/pull/353))

### v0.12.0 (2026-02-26)
- **Signal Channel**: Native Signal messaging via signal-cli HTTP daemon — first-class channel alongside Telegram with tool approval workflow, DM pairing, group support, and allowlist controls
- **OpenRouter Preset**: Setup wizard now includes OpenRouter as a dedicated provider option (200+ models via single API key)
- **Web UI — Tool Activity Cards**: Inline tool execution cards with animated spinner, elapsed timer, and auto-collapsing summary after response
- **Web UI — WASM Channel Setup Flow**: Improved setup stepper (Installed → Configured → Active) with state-aware action buttons
- **Web UI — Newest-First Logs**: Log viewer now displays most recent entries at the top
- **`--version` Flag**: `ironclaw --version` now officially supported, outputs `ironclaw 0.12.0`
- **Skills Enabled by Default**: Skills system now active by default with fixed registry and install pipeline
- **MCP Registry URL Fixes**: Corrected 6 MCP endpoint URLs, removed non-existent Google Drive and Google Calendar entries
- **Docker Build Fix**: Dockerfile now correctly copies `migrations/`, `registry/`, `channels-src/`, `wit/` directories
- **Extension Name Collision Fix**: Telegram and Slack tool registry names renamed to avoid conflicts with channel entries
- **Homebrew Install**: `brew install ironclaw` now available via `brew install ironclaw`

### v0.11.1 (2026-02-23)
- **CI/CD Fix**: Resolved release pipeline issue allowing custom `release.yml` jobs

### v0.11.0 (2026-02-23)
- **Context Auto-Compaction**: Automatic compaction with retry on `ContextLengthExceeded` — three strategies: Summarize (LLM-generated summary written to `daily/{date}.md`), Truncate (drop oldest turns), MoveToWorkspace (archive full turns)
- **Completion improvements**: Better handling of completion edge cases

### v0.10.0 (2026-02-22)
- **Smart Routing Provider**: Cost-optimized model selection — routes Simple queries to cheap models (e.g., Haiku), Complex queries to primary models (Sonnet/Opus), with cascade escalation on uncertain responses (`SMART_ROUTING_CASCADE`, `NEARAI_CHEAP_MODEL`)
- **Rate Limiting for Built-in Tools**: Per-tool, per-user sliding window rate limiting (per-minute and per-hour limits)
- **WASM Channel Enhancements**: Hot-activate WASM channels, channel-first prompts, unified artifact resolution
- **Pairing/Permission System**: All WASM channels now support device pairing and permissions
- **Group Chat Privacy**: Privacy controls and channel-aware prompts with safety hardening
- **Embedded Registry Catalog**: Offline-capable extension discovery with WASM bundle install pipeline
- **Token Usage & Cost Tracking**: Gateway status popover shows real-time token usage and cost
- **Custom HTTP Headers**: Support for `LLM_EXTRA_HEADERS` on OpenAI-compatible providers
- **HTML-to-Markdown Conversion**: New built-in tool for converting HTML content
- **FullJob Routine Mode**: Scheduler dispatch for routine jobs
- **Startup Optimization**: Startup time reduced from ~15s to ~2s
- **Web UI Refresh**: Agent-market design language, dashboard favicon, Chrome extension test skill

### v0.9.0 (2026-02-21)
- **TEE Attestation Shield**: Hardware-attested TEEs for enhanced security in web gateway UI
- **Configurable Tool Iterations**: New `AGENT_MAX_TOOL_ITERATIONS` setting for agentic loop control
- **Auto-Approve Tools**: New `AGENT_AUTO_APPROVE_TOOLS` for CI/benchmarking
- **X-Accel-Buffering**: SSE endpoint performance improvements

---

## Version

Documented: IronClaw v0.25.0
Release tag: [v0.25.0](https://github.com/nearai/ironclaw/releases/tag/v0.25.0) (2026-04-11)
Source: [github.com/nearai/ironclaw](https://github.com/nearai/ironclaw)
Docs repo: [github.com/mudrii/ironclaw-docs](https://github.com/mudrii/ironclaw-docs)
Generated: 2026-04-03
