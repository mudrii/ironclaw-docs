# IronClaw Developer Reference

> Version baseline: IronClaw v0.25.0 (`v0.25.0` tag snapshot)

Reference for developers building tools, channels, or contributing to IronClaw.

---

## Table of Contents

1. [CLI Commands — Tool Management](#1-cli-commands--tool-management)
2. [Tool Setup Schema](#2-tool-setup-schema)
3. [Environment Variables](#3-environment-variables)
4. [CI and Automated QA](#4-ci-and-automated-qa)

---

## 1. CLI Commands — Tool Management

The `ironclaw tool` subcommand manages WASM tools installed in `~/.ironclaw/tools/`.

### ironclaw tool install

Install a WASM tool from a source directory or a pre-built `.wasm` file.

```
ironclaw tool install <path> [options]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--name <name>` | directory/file name | Override the tool name |
| `--capabilities <path>` | auto-detected | Path to capabilities JSON file |
| `--target <dir>` | `~/.ironclaw/tools/` | Installation directory |
| `--release` | `true` | Build in release mode |
| `--skip-build` | false | Skip compilation, use existing `.wasm` |
| `--force` | false | Overwrite if tool already exists |

When `path` is a source directory, the tool looks for `Cargo.toml` and builds a WASM
component. When `path` is a `.wasm` file, it is copied directly.

### ironclaw tool list

List tools installed in `~/.ironclaw/tools/`.

```
ironclaw tool list [--dir <dir>] [--verbose]
```

`--verbose` shows path, hash (first 8 bytes), and capabilities summary for each tool.

### ironclaw tool remove

Remove an installed tool.

```
ironclaw tool remove <name> [--dir <dir>]
```

Deletes both the `.wasm` binary and the associated `.capabilities.json` file.

### ironclaw tool info

Show details for an installed tool or a `.wasm` file.

```
ironclaw tool info <name_or_path> [--dir <dir>]
```

Prints path, size, full SHA-256 hash, and a detailed capabilities breakdown including
allowed HTTP endpoints, secrets, workspace prefixes, and tool aliases.

### ironclaw tool auth

Configure OAuth or token authentication for a tool. Reads the `auth` section of
the tool's `capabilities.json`.

```
ironclaw tool auth <name> [--dir <dir>] [--user <user_id>]
```

`--user` defaults to `"default"`. The command supports three flows:
- **Environment variable**: detects the configured env var automatically
- **OAuth**: opens a browser for PKCE-based OAuth and exchanges the code for a token
- **Manual entry**: prompts for the token/API key directly

### ironclaw tool setup

Configure required secrets for a tool via its `setup.required_secrets` schema
(PR #438, added v0.13.0).

```
ironclaw tool setup <name> [--dir <dir>] [--user <user_id>]
```

`--user` defaults to `"default"`. The command reads the `setup` section of the tool's
`capabilities.json` and prompts the user for each entry in `required_secrets`. Each
secret is stored encrypted in the secrets store under the configured `name` key.

If a secret already exists, the user is asked whether to replace it. Optional secrets
can be skipped by pressing Enter without input.

Use `ironclaw tool setup` when a tool declares server-side credentials (e.g., OAuth
client IDs) via `setup.required_secrets`, and `ironclaw tool auth` when the user must
authenticate with a third-party service via `auth`.

### ironclaw models (v0.23.0)

Manage LLM providers and models from the CLI. Changes are persisted to `config.toml`
and `~/.ironclaw/.env`.

```
ironclaw models list [provider] [--verbose] [--json]
ironclaw models status [--json]
ironclaw models set <model>
ironclaw models set-provider <provider> [--model <model>]
```

| Subcommand | Description |
|------------|-------------|
| `list` | List all providers (active marked with `*`); optionally filter by provider |
| `status` | Show current active provider, model, fallback, and cheap model |
| `set` | Set the default model name (validates against known patterns) |
| `set-provider` | Set the LLM provider with optional model override |

Source: `src/cli/models.rs`.

### ironclaw hooks list (v0.23.0)

List discoverable lifecycle hooks from bundled and plugin (WASM capabilities) sources.
Workspace hooks require a DB connection and are not listed.

```
ironclaw hooks list [--verbose] [--json]
```

`--verbose` adds source, failure mode, and detailed hook points. `--json` outputs
machine-readable JSON. Hook kinds: `audit`, `rule`, `reject`, `webhook`. Hook
priority defaults: audit=25, rules=100, webhooks=300.

Source: `src/cli/hooks.rs`.

### ironclaw login (v0.23.0)

Re-authenticate with an LLM provider.

```
ironclaw login --openai-codex
```

Currently supports OpenAI Codex via device code OAuth. Tokens are persisted to
`~/.ironclaw/openai_codex_session.json`.

---

## 2. Tool Setup Schema

The `setup` section of a tool's `capabilities.json` declares secrets that must be
configured before the tool can operate. This schema is used by `ironclaw tool setup`.
The onboarding wizard's extension step installs tools and may suggest running
`ironclaw tool auth`, but does not execute `tool setup` automatically.

```json
{
  "setup": {
    "required_secrets": [
      {
        "name": "google_oauth_client_id",
        "prompt": "Google OAuth Client ID",
        "optional": false
      },
      {
        "name": "google_oauth_client_secret",
        "prompt": "Google OAuth Client Secret",
        "optional": true
      }
    ]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Key in the secrets store (e.g., `google_oauth_client_id`) |
| `prompt` | string | User-facing label shown during setup |
| `optional` | bool | If `true`, the user may skip this secret by pressing Enter |

The `setup` section is separate from the `auth` section. `setup` is for
infrastructure credentials (OAuth client IDs, webhook secrets) provided once by the
tool operator. `auth` is for per-user credentials obtained through an authentication
flow.

Source: `src/tools/wasm/capabilities_schema.rs` — `ToolSetupSchema`,
`ToolSecretSetupSchema`.

---

## 3. Environment Variables

### IRONCLAW_BASE_DIR

Overrides the IronClaw base directory (default: `~/.ironclaw`). Added in PR #397
(v0.13.0).

```bash
export IRONCLAW_BASE_DIR=/custom/ironclaw/path
ironclaw
```

The value is computed once at process startup and cached in a `std::sync::LazyLock`
for the lifetime of the process. Most runtime paths derived via base-dir helpers —
`~/.ironclaw/.env`, `~/.ironclaw/tools/`, and `~/.ironclaw/session.json` — use this
base directory. One exception in `v0.15.0`: libSQL auto-detection still checks the
default `~/.ironclaw/ironclaw.db` path directly in bootstrap.

| Behavior | Description |
|----------|-------------|
| Not set | Uses `~/.ironclaw` (or `./.ironclaw` if home dir cannot be determined) |
| Set to an absolute path | Uses that path |
| Set to a relative path | Issues a warning and uses the path relative to the current directory |
| Set to empty string | Treated as unset; falls back to default |

Source: `src/bootstrap.rs` — `ironclaw_base_dir()`, `IRONCLAW_BASE_DIR` constant.

### Multi-Tenant Environment Variables (v0.23.0)

Multi-tenant mode is auto-detected when `GATEWAY_USER_TOKENS` is set. These env vars
control per-user resource limits in multi-tenant deployments.

| Variable | Default | Description |
|----------|---------|-------------|
| `TENANT_MAX_LLM_CONCURRENT` | `4` | Max concurrent LLM calls per user |
| `TENANT_MAX_JOBS_CONCURRENT` | `3` | Max concurrent jobs per user |
| `HEARTBEAT_MULTI_TENANT` | auto | Cycle heartbeat through all users (auto from `GATEWAY_USER_TOKENS`) |
| `MAX_COST_PER_USER_PER_DAY_CENTS` | unlimited | Per-user daily LLM spend cap in cents |

### New LLM Provider Environment Variables (v0.23.0)

**GitHub Copilot:**

| Variable | Default | Description |
|----------|---------|-------------|
| `GITHUB_COPILOT_TOKEN` | — | Copilot auth token (required for `github_copilot` backend) |
| `GITHUB_COPILOT_EXTRA_HEADERS` | — | Custom headers (`Key:Value,Key:Value`) for Copilot requests |

**OpenAI Codex (ChatGPT subscription):**

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENAI_CODEX_MODEL` | `gpt-5.3-codex` | Model name |
| `OPENAI_CODEX_API_URL` | `https://chatgpt.com/backend-api/codex` | API base URL |
| `OPENAI_CODEX_AUTH_URL` | `https://auth.openai.com` | OAuth auth server |
| `OPENAI_CODEX_CLIENT_ID` | `app_EMoamEEZ73f0CkXaXp7hrann` | OAuth client ID |
| `OPENAI_CODEX_SESSION_PATH` | `~/.ironclaw/openai_codex_session.json` | Token file |
| `OPENAI_CODEX_REFRESH_MARGIN_SECS` | `300` | Pre-emptive token refresh margin |

**Gemini:**

| Variable | Default | Description |
|----------|---------|-------------|
| `GEMINI_MODEL` | `gemini-2.5-flash` | Model name |
| `GEMINI_CREDENTIALS_PATH` | `~/.gemini/oauth_creds.json` | OAuth credentials file |

### Embedding Cache (v0.23.0)

| Variable | Default | Description |
|----------|---------|-------------|
| `EMBEDDING_CACHE_SIZE` | `10000` | LRU cache entries for embeddings (~58 MB at 1536-dim) |

---

## 4. CI and Automated QA

The CI pipeline runs on every pull request and push to `main`. It is defined in
`.github/workflows/test.yml` and covers three parallel test jobs plus a Docker build
(PR #353, v0.13.0).

### Test matrix

| Job name | Cargo flags | Purpose |
|----------|-------------|---------|
| `all-features` | `--all-features` | Full feature set including postgres and libsql |
| `default` | (none) | Default features |
| `libsql-only` | `--no-default-features --features libsql` | libsql-only, no postgres |

Each job runs `cargo test $flags -- --nocapture`.

### Telegram channel tests

A separate job compiles and tests the Telegram channel crate independently:

```
cargo test --manifest-path channels-src/telegram/Cargo.toml -- --nocapture
```

### Docker build

A `docker-build` job runs `docker build -t ironclaw-test:ci .` to verify the
Dockerfile compiles cleanly. This validates the container build path independently
of the host Rust toolchain.

### Roll-up gate

All four jobs (`tests`, `telegram-tests`, `docker-build`, and the roll-up `run-tests`)
must pass. The `run-tests` roll-up job is used as the branch protection target. A
pull request cannot be merged if any of the three underlying jobs fail.

---

## 5. Trace Recording and Replay (v0.16.0)

IronClaw v0.16.0 introduced a live trace recording system for creating deterministic E2E test fixtures without a live LLM or network.

### Capturing a trace

Run any real session with `IRONCLAW_RECORD_TRACE` set:

```bash
IRONCLAW_RECORD_TRACE=/tmp/my-session.json ironclaw run
```

Every LLM request, tool call, and HTTP exchange is written to the JSON file. The agent runs normally during recording — all real requests go through.

### Trace file format

```json
{
  "model_name": "gpt-4o",
  "memory_snapshot": [
    { "path": "identity/MEMORY.md", "content": "..." }
  ],
  "http_exchanges": [
    {
      "request":  { "method": "GET", "url": "https://api.example.com/data", "headers": [], "body": null },
      "response": { "status": 200, "headers": [], "body": "{\"result\": 42}" }
    }
  ],
  "steps": [
    {
      "input": "What is 6×7?",
      "expected": { "type": "text", "content": "42" }
    }
  ]
}
```

### Replaying a trace in tests

Use `TraceLlm` in the test rig:

```rust
let app = AppBuilder::new()
    .with_llm(TraceLlm::from_file("tests/fixtures/my-session.json"))
    .build()
    .await?;
```

During replay:
- `TraceLlm` returns pre-recorded LLM responses step by step.
- `HttpInterceptor` intercepts outgoing HTTP calls and returns the pre-recorded responses — no real network requests.
- Workspace is pre-seeded from `memory_snapshot`.

### HTTP interception architecture

`JobContext` carries an `http_interceptor: Option<Arc<dyn HttpInterceptor>>`. The built-in `http` tool checks this before sending real requests. Tools call `interceptor.before_request(&req).await` — if `Some(response)` is returned, the real request is skipped.

This makes all HTTP-dependent tests hermetic and reproducible without mocking at the network layer.

### Fixture directory

Pre-recorded traces are stored in `tests/fixtures/llm_traces/`. The directory is organized by scenario type:

| Subdirectory | Purpose |
|---|---|
| `spot/` | Smoke tests — greeting, math, echo, tool |
| `coverage/` | Per-tool coverage scenarios |
| `advanced/` | Multi-turn, memory, steering, iteration-limit |
| `recorded/` | Real-world sessions (weather, baseball stats) |
| `worker/` | Worker/orchestrator scenarios |
| `workspace/` | Workspace search and document lifecycle |
| `tools/` | Individual tool traces (http, jobs, routines) |
