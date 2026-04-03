# IronClaw v0.23.0 — LLM Backend System Deep Dive

> **Scope:** `src/llm/`, `src/config/llm.rs`, `src/agent/cost_guard.rs`,
> `src/agent/context_monitor.rs`, `src/agent/compaction.rs`,
> `src/estimation/`, `src/evaluation/`, `src/observability/`

---

## Table of Contents

1. [Overview](#1-overview)
2. [Provider Trait and Core Types](#2-provider-trait-and-core-types)
3. [Supported Backends](#3-supported-backends)
4. [Reliability Wrapper Chain](#4-reliability-wrapper-chain)
5. [Smart Routing Provider](#5-smart-routing-provider)
6. [NEAR AI Provider — Consolidated Chat Completions](#6-near-ai-provider--consolidated-chat-completions)
7. [rig-core Adapter and Schema Normalization](#7-rig-core-adapter-and-schema-normalization)
8. [GitHub Copilot Provider](#8-github-copilot-provider)
9. [OpenAI Codex Provider (ChatGPT Subscription)](#9-openai-codex-provider-chatgpt-subscription)
10. [Gemini OAuth Provider (Cloud Code API)](#10-gemini-oauth-provider-cloud-code-api)
11. [Per-Tool Reasoning System](#11-per-tool-reasoning-system)
12. [Session Management](#12-session-management)
13. [Configuration Resolution](#13-configuration-resolution)
14. [Cost Accounting and Guardrails](#14-cost-accounting-and-guardrails)
15. [Context Window Management](#15-context-window-management)
16. [Estimation, Evaluation, and Observability](#16-estimation-evaluation-and-observability)
17. [Extension Points](#17-extension-points)

---

## 1. Overview

IronClaw's LLM subsystem is structured as a layered stack with three tiers:

```
┌─────────────────────────────────────────────────────────┐
│  Reasoning  (src/llm/reasoning.rs)                      │
│  Orchestrates plan → select_tools → respond_with_tools  │
├─────────────────────────────────────────────────────────┤
│  Reliability wrappers (decorator chain)                 │
│  SmartRoutingProvider → RetryProvider →                 │
│  CircuitBreakerProvider → ResponseCacheProvider →       │
│  FailoverProvider → actual backend                      │
├─────────────────────────────────────────────────────────┤
│  Backend providers                                      │
│  NearAiChatProvider │ RigAdapter<M> │                   │
│  GithubCopilotProvider │ OpenAiCodexProvider │          │
│  GeminiOauthProvider │ CodexChatGptProvider             │
│  (OpenAI, Anthropic, Ollama, OpenAI-compatible,         │
│   Tinfoil, Bedrock)                                     │
└─────────────────────────────────────────────────────────┘
```

All tiers implement the same `LlmProvider` trait (`src/llm/provider.rs`), so
the `Reasoning` orchestrator is blind to which backend or combination of
wrappers sits beneath it. Reliability behaviors compose by wrapping — adding
retry logic never requires modifying a provider implementation.

**Changes since v0.19.0:**

- Added GitHub Copilot provider with device-login OAuth and two-step token exchange
- Added OpenAI Codex provider using the Responses API with ChatGPT subscription OAuth
- Added Gemini OAuth provider integrating Google Cloud Code API
- Added per-tool reasoning threading through the provider chain
- Added XML tool-call recovery with context-aware filtering (code blocks, blockquotes, inline code)
- Added `f32` to `f64` temperature precision fix to eliminate provider 400 errors
- Provider registry replaced the `LlmBackend` enum for most providers
- MiniMax default model updated from `MiniMax-M2.5` to `MiniMax-M2.7`
- Total backend count: **15+** (NearAI, OpenAI, Anthropic, Ollama, OpenAI-compatible, Tinfoil, MiniMax, Z.AI, Codex/ChatGPT, GitHub Copilot, OpenAI Codex, Gemini OAuth, Bedrock, plus any OpenAI-compatible endpoint)

---

## 2. Provider Trait and Core Types

### 2.1 The `LlmProvider` Trait

Defined in `src/llm/provider.rs`. Every provider — native or wrapped — must
implement these methods:

```rust
#[async_trait]
pub trait LlmProvider: Send + Sync {
    async fn complete(&self, req: CompletionRequest) -> Result<CompletionResponse, LlmError>;
    async fn complete_with_tools(
        &self,
        req: ToolCompletionRequest,
    ) -> Result<ToolCompletionResponse, LlmError>;
    fn model_name(&self) -> &str;
    fn cost_per_token(&self) -> (Decimal, Decimal); // (input, output) per token
    fn active_model_name(&self) -> String;
    fn set_model(&self, model: &str) -> Result<(), LlmError>;
    async fn list_models(&self) -> Result<Vec<String>, LlmError>;
    async fn model_metadata(&self) -> Result<ModelMetadata, LlmError>;
    fn effective_model_name(&self, requested_model: Option<&str>) -> String;
    fn calculate_cost(&self, input_tokens: u32, output_tokens: u32) -> Decimal;
}
```

The default `calculate_cost()` implementation multiplies token counts by the
rates from `cost_per_token()`:

```rust
fn calculate_cost(&self, input_tokens: u32, output_tokens: u32) -> Decimal {
    let (input_rate, output_rate) = self.cost_per_token();
    input_rate * Decimal::from(input_tokens) + output_rate * Decimal::from(output_tokens)
}
```

### 2.2 Message and Completion Types

| Type | Purpose |
|------|---------|
| `ChatMessage` | Role-tagged message (`System`, `User`, `Assistant`, `Tool`) |
| `CompletionRequest` | Messages + max_tokens + temperature + stop_sequences |
| `CompletionResponse` | Content string + `FinishReason` + `TokenUsage` |
| `ToolCompletionRequest` | Extends `CompletionRequest` with `Vec<ToolDefinition>` |
| `ToolCompletionResponse` | Either `Text(String)` or `ToolCalls { calls, content }` |
| `ToolDefinition` | Name + description + JSON Schema parameters |
| `ToolCall` | ID + name + arguments (JSON `Value`) + optional `reasoning` |
| `ToolResult` | ID + content (for feeding results back) |

### 2.3 Tool Message Sanitization

A key correctness concern: providers like Anthropic return `HTTP 400` when a
conversation references a `tool_call_id` that was never issued in the current
context (e.g., after failover to a different provider mid-session).

`sanitize_tool_messages()` in `provider.rs` scans the message list and rewrites
orphaned `role: Tool` messages as plain user messages:

```rust
// Before: Tool { tool_call_id: "abc", content: "result" }
// After:  User { content: "Tool result: result" }
```

This prevents HTTP 400 errors when switching providers mid-conversation.

---

## 3. Supported Backends

### 3.1 Provider Registry

In v0.23.0, most providers are resolved through a **provider registry** (`src/llm/registry.rs`) using `ProviderProtocol` to select the appropriate rig-core client constructor:

```rust
pub enum ProviderProtocol {
    OpenAiCompletions, // OpenAI, Tinfoil, Groq, NVIDIA NIM, OpenRouter, etc.
    Anthropic,         // Anthropic Messages API
    Ollama,            // Ollama API (no API key)
    GithubCopilot,     // GitHub Copilot (token exchange)
}
```

Special backends handled outside the registry:
- `nearai` — uses `NearAiChatProvider` (dual auth: session token or API key)
- `gemini_oauth` — uses `GeminiOauthProvider` (Google Cloud Code API)
- `bedrock` — uses `BedrockProvider` (AWS Converse API, feature-gated)
- `openai_codex` — uses `OpenAiCodexProvider` (Responses API, subscription OAuth)

### 3.2 Factory in `src/llm/mod.rs`

`create_llm_provider(config)` dispatches on `config.backend` to select the
appropriate provider path, then `build_provider_chain()` wraps it in the
reliability decorator chain.

Notable factory quirks:

- **OpenAI uses `CompletionsClient`** (not the Responses API client) to avoid
  a known `rig-core` panic when threading `call_id` values.
- **Tinfoil** always targets `https://inference.tinfoil.sh/v1`, supports only
  Chat Completions (no Responses API), and adapts tool calls to Chat-format.
- **Ollama** defaults to `http://localhost:11434` and the `llama3` model.
- **GitHub Copilot** uses a dedicated `GithubCopilotProvider` with direct HTTP
  (not rig-core) because Copilot requires two-step token exchange.
- **OpenAI Codex** uses a dedicated factory path via `build_provider_chain()`.

### 3.3 Provider Matrix

| Backend | Auth | API Style | Tool Support | Cost Tracking |
|---------|------|-----------|--------------|---------------|
| NearAiChat (API key mode) | API key (`NEARAI_API_KEY`) | Chat Completions | Flattened to text | Via cost table |
| NearAiChat (session mode) | Session token (`SessionManager`) | Chat Completions | Flattened to text | Via cost table |
| OpenAI | API key | Chat Completions | Native (strict schema) | Via cost table |
| Anthropic | API key or OAuth | Anthropic API | Native | Via cost table |
| Ollama | None | Chat Completions | Native | Zero cost (local) |
| OpenAiCompatible | Optional key | Chat Completions | Native | Via cost table |
| Tinfoil | API key | Chat Completions | Chat-format | Via cost table |
| MiniMax | `MINIMAX_API_KEY` (via registry) | Chat Completions | Native | Via cost table | default model: `MiniMax-M2.7` |
| Z.AI | `ZAI_API_KEY` (via registry) | Chat Completions | Native | Via cost table | default model: `glm-4-plus` |
| Codex / ChatGPT | OpenAI + `LLM_USE_CODEX_AUTH=true` | Codex CLI token | No separate API key; reads `~/.codex/auth.json` | Via cost table |
| GitHub Copilot | `GITHUB_COPILOT_TOKEN` | Chat Completions (token exchange) | Native | Via cost table |
| OpenAI Codex | `ironclaw login --openai-codex` | Responses API (SSE) | Native | Zero cost (subscription) |
| Gemini OAuth | Google OAuth (Cloud Code) | Gemini generateContent | Native (functionDeclarations) | Zero cost (subscription) |
| AWS Bedrock | IAM / SSO | Converse API | Native | Via cost table |

---

## 4. Reliability Wrapper Chain

The reliability wrappers in `src/llm/` all implement `LlmProvider` and
accept `Arc<dyn LlmProvider>` as their inner provider. They compose cleanly:

```
SmartRoutingProvider(
  RecordingLlm(          <- trace capture layer (when IRONCLAW_RECORD_TRACE=1)
    RetryProvider(
      CircuitBreakerProvider(
        CachedProvider(
          FailoverProvider([primary, fallback])
        )
      )
    )
  )
)
```

The `RecordingLlm` wrapper is **only** present when `IRONCLAW_RECORD_TRACE` is set; otherwise the chain collapses back to the four-layer form.

### 4.0 Trace Recording — `recording.rs` (v0.16.0)

`RecordingLlm` wraps any `LlmProvider` and captures every interaction into the JSON
trace fixture format used by `TraceLlm` for deterministic E2E testing. Enable at
runtime by setting `IRONCLAW_RECORD_TRACE=<output-path>`.

**What is recorded:**

| Component | Detail |
|-----------|--------|
| `memory_snapshot` | Workspace documents captured before the first LLM call |
| `http_exchanges` | All outgoing HTTP request/response pairs from tools (via `HttpInterceptor`) |
| `steps` | User inputs, LLM responses (text + tool_calls), expected tool results |

**Trace file format** (`TraceFile`):

```json
{
  "model_name": "gpt-4o",
  "memory_snapshot": [
    { "path": "identity/MEMORY.md", "content": "..." }
  ],
  "http_exchanges": [
    {
      "request":  { "method": "GET", "url": "...", "headers": [], "body": null },
      "response": { "status": 200, "headers": [], "body": "..." }
    }
  ],
  "steps": [
    { "input": "What is the weather?", "expected": { "type": "tool_calls", "calls": [...] } }
  ]
}
```

Recorded traces are written to the path specified in `IRONCLAW_RECORD_TRACE` (must end with `.json`). They can be replayed via `TraceLlm` in tests without network access — `HttpInterceptor` returns pre-recorded responses instead of making real HTTP requests.

**HTTP interception** is wired into `JobContext.http_interceptor`. The `http` built-in tool checks this interceptor before sending real requests; during replay it returns the pre-recorded response.

```env
IRONCLAW_RECORD_TRACE=/tmp/my-trace.json  # Set to any .json path to enable recording
```

### 4.1 Retry — `retry.rs`

**Retryable errors** (will retry): `RequestFailed`, `RateLimited`,
`InvalidResponse`, `SessionRenewalFailed`, `Http`, `Io`.

**Non-retryable errors** (fail immediately): `AuthFailed`, `SessionExpired`,
`ContextLengthExceeded`, `ModelNotAvailable`, `Json`.

**Backoff formula:**

```
delay = base_ms * 2^attempt * jitter_factor
base_ms = 1000ms
jitter_factor = random in [0.75, 1.25]  // +/-25%
minimum_delay = 100ms (floor)
```

For attempt 0: ~750-1250ms. Attempt 1: ~1500-2500ms. Attempt 2: ~3000-5000ms.

**Rate limit hint:** If the error is `RateLimited { retry_after: Some(dur) }`,
the retry provider sleeps exactly `dur` instead of computing backoff.

Default `max_retries` is 3, giving 4 total attempts (1 initial + 3 retries).

### 4.2 Circuit Breaker — `circuit_breaker.rs`

**State machine:**

```
Closed --(threshold consecutive transient failures)--> Open
Open   --(recovery_timeout elapsed)--------------------> HalfOpen
HalfOpen --(half_open_successes_needed successes)------> Closed
HalfOpen --(any failure)-------------------------------> Open
```

**Key distinction from retry:** The circuit breaker includes `SessionExpired`
in its `is_transient()` check, while `retry.rs::is_retryable()` does not.
This means session expiry counts toward tripping the circuit breaker (signaling
persistent auth problems) but does not trigger individual retries.

**Default parameters:**

- `failure_threshold`: 5 consecutive transient failures
- `recovery_timeout`: 30 seconds
- `half_open_successes_needed`: 2 successful probes before fully closing

When Open, all requests are immediately rejected with `CircuitOpen` error
without touching the downstream provider.

### 4.3 Response Cache — `response_cache.rs`

`CachedProvider` intercepts `complete()` calls only. Tool calls via
`complete_with_tools()` are **never cached** because they may have side
effects.

**Cache key:** SHA-256 of `(model_name, messages_json, max_tokens, temperature,
stop_sequences)`. Any change to any field produces a different key.

**Eviction policy:** LRU by `last_accessed` timestamp. On each write, expired
entries (TTL exceeded) are pruned first, then the oldest entry is evicted if
at capacity.

**Default parameters:**

- `response_cache_enabled`: false (opt-in)
- `response_cache_ttl_secs`: 3600 (1 hour)
- `response_cache_max_entries`: 1000

### 4.4 Failover — `failover.rs`

`FailoverProvider` holds an ordered `Vec` of providers and tries each in
sequence when the active provider produces a retryable error.

**Cooldown mechanism:** Each provider tracks consecutive retryable failures
using lock-free atomics (`AtomicU32` failure count, `AtomicI64` cooldown
start time). A provider in cooldown is skipped. If all providers are in
cooldown, the one with the oldest cooldown start time is selected (never
blocks entirely).

**Per-task provider tracking:** Under concurrent requests, different tasks may
be using different providers in the failover chain simultaneously.
`provider_for_task: Mutex<HashMap<tokio::task::Id, usize>>` stores which
provider index each tokio task is currently using, so follow-up calls from
the same logical request stay on the same provider.

**Default parameters:**

- `failover_cooldown_secs`: 300 (5 minutes)
- `failover_cooldown_threshold`: 3 consecutive failures before cooldown

---

## 5. Smart Routing Provider

**`src/llm/smart_routing.rs`** (1 745 lines) — Redesigned in v0.16.0 (#529)

`SmartRoutingProvider` implements cost-optimized model selection by analyzing prompt complexity across 13 dimensions and routing to the appropriate model tier.

The provider wraps two `LlmProvider` instances and implements the trait itself, fitting into the standard provider chain:

```
SmartRoutingProvider -> RetryProvider -> CircuitBreakerProvider -> ResponseCacheProvider -> FailoverProvider -> actual backend
          |
   +------+------+
   |              |
Cheap Model  Primary Model
```

### 5.1 Complexity Tiers

The scorer produces a 0-100 score mapped to four tiers via `Tier::from_score()`:

| Score | Tier | Typical Use Cases |
|-------|------|------------------|
| 0-15 | `Flash` | Greetings, quick lookups (time, date, weather) |
| 16-40 | `Standard` | Writing, comparisons, defined tasks |
| 41-65 | `Pro` | Multi-step analysis, code review |
| 66+ | `Frontier` | Security audits, critical decisions |

These four tiers are mapped to the three `TaskComplexity` variants used by the router:

| Tier(s) | `TaskComplexity` | Destination |
|---------|-----------------|-------------|
| `Flash`, `Standard` | `Simple` | Cheap model |
| `Pro` | `Moderate` | Cheap model with cascade escalation |
| `Frontier` | `Complex` | Primary model always |

> **Tool use always routes to the primary model** regardless of tier, since structured JSON output requires reliable generation.

### 5.2 Fast-Path Pattern Overrides

Before scoring, `classify()` checks a set of compiled `PatternOverride` regexes that short-circuit the scorer for obvious cases:

| Pattern | Tier |
|---------|------|
| `^(hi|hello|hey|thanks|ok|sure|yes|no|yep|nope|cool|nice|great|got it)$` | Flash |
| `^what(?:'s|\s+is)?\s+(?:the\s+)?(time\|date\|day\|weather)\b...` | Flash |
| `security.*(audit\|review\|scan)` | Frontier |
| `vulnerabilit(y\|ies).*(review\|scan\|check\|audit)` | Frontier |
| `deploy.*(mainnet\|production)` | Pro |
| `production.*(deploy\|release\|push)` | Pro |

### 5.3 13-Dimension Scorer

`score_complexity()` / `score_complexity_with_config()` — computes a weighted sum across 13 independently scored dimensions:

| Dimension | Default Weight | Key Signals |
|-----------|--------------|-------------|
| `reasoning_words` | 14% | "why", "explain", "compare", "trade-offs" |
| `token_estimate` | 12% | Prompt length in tokens |
| `code_indicators` | 10% | Backticks, `implement`, `PR`, `refactor` |
| `multi_step` | 10% | "first", "then", "after", numbered steps |
| `domain_specific` | 10% | Technical terms (configurable via `domain_keywords`) |
| `creativity` | 7% | "write", "summarize", "tweet", "blog" |
| `question_complexity` | 7% | Multiple questions, open-ended starters |
| `precision` | 6% | Numbers, "exactly", "calculate" |
| `ambiguity` | 5% | Vague references |
| `context_dependency` | 5% | "previous", "you said" |
| `sentence_complexity` | 5% | Commas, conjunctions, clause depth |
| `tool_likelihood` | 5% | "read", "deploy", "install" |
| `safety_sensitivity` | 4% | "password", "auth", "vulnerability" |

**Multi-dimensional boost:** if 3 or more dimensions score above their threshold, the total receives a +30% additive boost.

**Explicit tier hints:** prompts containing `[tier:flash]`, `[tier:standard]`, `[tier:pro]`, or `[tier:frontier]` (case-insensitive) bypass scoring entirely.

### 5.4 Cascade Mode

Controlled by `SMART_ROUTING_CASCADE` (default: `true`).

When enabled, `Moderate` (`Pro`-tier) requests go to the cheap model first. If the cheap model returns an uncertain response, the request is automatically escalated to the primary model.

Uncertainty is detected by scanning the response content for any of these phrases:
- `"i'm not sure"` / `"i am not sure"`
- `"i don't know"` / `"i do not know"`
- `"i'm unable to"` / `"i am unable to"`
- `"i cannot"` / `"i can't"`
- `"beyond my capabilities"` / `"beyond my ability"`
- `"i need more context"` / `"i need more information"`
- `"could you clarify"` / `"could you provide more"`
- `"i'm not confident"` / `"i am not confident"`
- Empty response (zero-length content)

### 5.5 Configuration

```
SMART_ROUTING_CASCADE=true         # Enable cascade escalation for Pro-tier tasks (default: true)
NEARAI_CHEAP_MODEL=...             # Cheap model name; smart routing disabled if unset
LLM_CHEAP_MODEL=...                # Generic cheap model (overrides NEARAI_CHEAP_MODEL, works with any backend)
```

If no cheap model is configured, smart routing is disabled and all requests go to the primary model.

Custom domain keywords can be injected via `ScorerConfig::domain_keywords` (programmatic API — no env var).

### 5.6 Observable Statistics

`SmartRoutingProvider::stats()` returns a `SmartRoutingSnapshot` with these atomic counters:

| Counter | Description |
|---------|-------------|
| `total_requests` | All requests processed by the provider |
| `cheap_requests` | Requests routed to the cheap model |
| `primary_requests` | Requests routed to the primary model |
| `cascade_escalations` | Pro-tier responses escalated to primary due to uncertainty |

---

## 6. NEAR AI Provider — Consolidated Chat Completions (`src/llm/nearai_chat.rs`)

In v0.9.0, the separate `nearai.rs` (Responses API) was removed. Both auth modes are now unified in `nearai_chat.rs` using the Chat Completions API.

### 6.1 Auth Modes

| Mode | Trigger | Base URL | Token source |
|------|---------|----------|--------------|
| API key | `NEARAI_API_KEY` is set | `https://cloud-api.near.ai` (default) | API key as Bearer token |
| Session token | No `NEARAI_API_KEY` | `https://private.near.ai` (default) | `SessionManager` (auto-renew on 401) |

Both modes hit the same endpoint: `{NEARAI_BASE_URL}/v1/chat/completions`

The `NearAiChatProvider` struct holds: `client`, `config`, `session: Arc<SessionManager>`, `active_model: RwLock<String>`, `flatten_tool_messages: bool`. Auth is resolved via `resolve_bearer_token()`, which picks the API key or session token depending on which mode is active.

### 6.2 Tool Message Flattening

NEAR AI does not support `role: "tool"` messages. The provider rewrites them via `flatten_tool_messages()` before sending:

- Assistant messages with `tool_calls` -> plain assistant text: `[Called tool \`name\` with arguments: {...}]`
- Tool result messages (`role: "tool"`) -> user messages: `[Tool \`name\` returned: result]`

### 6.3 Session Token Renewal

On 401 response (session token mode only):

1. Check if body contains "session" + ("expired" | "invalid")
2. If yes -> `LlmError::SessionExpired` -> `session.handle_auth_failure()` -> retry once
3. If no -> `LlmError::AuthFailed`

Note: retries on other errors are handled by the outer `RetryProvider` wrapper, not here.

### 6.4 Model Listing

`GET /v1/models` — handles flexible response formats:

- `{models: [...]}` or `{data: [...]}` (OpenAI-style)
- Direct array `[...]`
- Each entry: tries `name`, `id`, `model`, `model_name`, `model_id`, nested `metadata.name`

### 6.5 Usage Parsing Resilience

Some providers omit `completion_tokens` from the usage block. The `parse_usage()` function handles this by computing `completion_tokens = total_tokens - prompt_tokens` when `completion_tokens` is missing.

### 6.6 Reasoning Content Handling (v0.16.0, PR #580)

Some reasoning models (e.g. GLM-5) return their answer in `reasoning_content` rather than `content` — with `content: null`. The provider handles this with a guarded fallback:

- For **text responses** (no tool calls): use `content`, fall back to `reasoning_content` if `content` is null.
- For **tool-call responses**: always use `content`; **never** fall back to `reasoning_content`.

This prevents chain-of-thought reasoning tokens from leaking into conversation history during tool calls, which would inflate the context window and confuse subsequent turns.

```rust
let content = if tool_calls.is_empty() {
    choice.message.content.or(choice.message.reasoning_content)
} else {
    choice.message.content  // reasoning_content intentionally ignored
};
```

---

## 7. rig-core Adapter and Schema Normalization

`src/llm/rig_adapter.rs` bridges `rig-core`'s `CompletionModel` trait to
IronClaw's `LlmProvider` trait. This adapter is used by OpenAI, Anthropic,
Ollama, OpenAI-compatible, and Tinfoil backends.

### 7.1 OpenAI Strict Mode Schema Normalization

OpenAI's strict mode requires JSON Schemas to be fully specified with no
ambiguity. `normalize_schema_strict()` transforms tool parameter schemas:

1. Sets `"additionalProperties": false` on all object types
2. Moves all properties to `"required"` array
3. Makes originally-optional properties nullable by wrapping their type in
   `["type", "null"]` via `make_nullable()`

This normalization happens recursively for nested object schemas. Without it,
OpenAI returns schema validation errors for tools with optional parameters.

### 7.2 Message Conversion

`convert_messages()` splits the flat `Vec<ChatMessage>` into rig-core's
expected structure:

- System role messages -> `preamble` (rig-core uses this separately)
- All other messages -> `history` (chronological order)

### 7.3 Tool Name and ID Normalization

**Tool name:** `normalize_tool_name()` strips the `proxy_` prefix that some
OpenAI-compatible proxies add to tool names (e.g., LiteLLM wrapping tools).
Without this, tool dispatch fails to find the registered tool.

**Tool call ID:** `normalized_tool_call_id()` generates a fallback ID
`generated_tool_call_{seed}` for empty or missing tool call IDs, which
some providers omit.

### 7.4 Temperature Precision Fix (PR #1450)

The `round_f32_to_f64()` function prevents floating-point precision artifacts
when converting IronClaw's `f32` temperature values to `f64` for the rig-core
API. Direct `f32 as f64` cast preserves the binary representation, producing
values like `0.699999988079071` instead of `0.7`. Some providers (e.g.
Zhipu/GLM) reject these values with a 400 error.

The function rounds to 6 decimal places after casting, removing the artifact
while preserving all meaningful precision:

```rust
fn round_f32_to_f64(val: f32) -> f64 {
    ((val as f64) * 1_000_000.0).round() / 1_000_000.0
}
```

---

## 8. GitHub Copilot Provider

**`src/llm/github_copilot.rs`** + **`src/llm/github_copilot_auth.rs`**

### 8.1 Architecture

The GitHub Copilot API at `api.githubcopilot.com` speaks OpenAI Chat Completions
format but requires a two-step authentication flow that prevents using the
standard rig-core OpenAI client:

1. A long-lived GitHub OAuth token (from device login or IDE sign-in)
2. A short-lived Copilot session token (exchanged via `api.github.com/copilot_internal/v2/token`)

`GithubCopilotProvider` uses direct HTTP via `reqwest::Client` with a
`CopilotTokenManager` that caches the session token and refreshes it
automatically 5 minutes before expiry.

### 8.2 Authentication Flow

**Device login flow** (`github_copilot_auth.rs`):

1. `request_device_code()` — POST to `github.com/login/device/code` with VS Code Copilot client ID (`Iv1.b507a08c87ecfe98`)
2. User enters the code at the verification URL
3. `poll_for_access_token()` — polls `github.com/login/oauth/access_token` until authorized
4. `validate_token()` — exchanges OAuth token for Copilot session token, verifies against `/models` endpoint

**Token exchange** (`CopilotTokenManager`):
- `get_token()` — returns cached session token if valid (>5min remaining), otherwise exchanges via `copilot_internal/v2/token`
- `invalidate()` — clears cache on 401, forcing fresh exchange on next call
- Uses `RwLock` for concurrent reads, upgrades to write lock only for refresh

**Identity headers** injected into every request:
- `User-Agent: GitHubCopilotChat/0.26.7`
- `Editor-Version: vscode/1.99.3`
- `Editor-Plugin-Version: copilot-chat/0.26.7`
- `Copilot-Integration-Id: vscode-chat`

### 8.3 Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GITHUB_COPILOT_TOKEN` | -- | Required. GitHub OAuth token (from device login or IDE) |
| `GITHUB_COPILOT_MODEL` | -- | Model identifier |
| `GITHUB_COPILOT_EXTRA_HEADERS` | -- | Additional headers injected into requests |

**LLM_BACKEND value:** `github_copilot`

### 8.4 Known Risks

The device login flow uses VS Code Copilot's OAuth client ID and injects
VS Code identity headers. GitHub could rotate this client ID or reject stale
editor version strings at any time. If GitHub publishes an official third-party
client ID or OAuth app registration flow, it should be adopted immediately.

---

## 9. OpenAI Codex Provider (ChatGPT Subscription)

**`src/llm/openai_codex_provider.rs`** + **`src/llm/openai_codex_session.rs`** + **`src/llm/token_refreshing.rs`**

### 9.1 Architecture

The OpenAI Codex provider uses the Responses API at
`chatgpt.com/backend-api/codex/responses` with ChatGPT subscription OAuth tokens.
This is subscription-based billing (zero API cost).

**Provider chain:** `OpenAiCodexProvider` -> `TokenRefreshingProvider` -> standard decorator chain

### 9.2 Authentication Flow

**Device code flow** (`openai_codex_session.rs`):

1. POST `/api/accounts/deviceauth/usercode` -> get `device_auth_id` + `user_code`
2. User enters code at `auth.openai.com/codex/device`
3. Poll POST `/api/accounts/deviceauth/token` -> get `authorization_code` + PKCE
4. Exchange via POST `/oauth/token` -> get `access_token` + `refresh_token`

Initiated via `ironclaw login --openai-codex`.

**Session persistence:** Tokens saved to `~/.ironclaw/openai_codex_session.json` (mode `0600`). Auto-refreshed before expiry with a 300-second margin.

**JWT account ID extraction:** The `chatgpt_account_id` is extracted from the JWT payload at `["https://api.openai.com/auth"]["chatgpt_account_id"]` and included as a header in every request.

### 9.3 Token Refreshing Decorator

`TokenRefreshingProvider` wraps `OpenAiCodexProvider`:
- Pre-emptively refreshes OAuth token before each call if near expiry
- Updates the inner provider's token after refresh (no client rebuild needed)
- Retries once on `AuthFailed` / `SessionExpired` after refreshing
- Overrides `cost_per_token()` to return `(0, 0)` — subscription billing

### 9.4 Responses API Integration

The provider uses SSE streaming with Responses API event types:
- `response.output_text.delta` — text content streaming
- `response.output_item.added` / `response.output_item.done` — function call lifecycle
- `response.function_call_arguments.delta` / `.done` — streaming tool call arguments
- `response.completed` — usage metadata and status
- `response.failed` / `error` — error handling

System messages are sent as the `instructions` field, not in the `input` array.
Tool schemas are normalized via `normalize_schema_strict()` for OpenAI strict mode.

### 9.5 Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENAI_CODEX_MODEL` | `gpt-5.3-codex` | Model identifier |
| `OPENAI_CODEX_API_URL` | `https://chatgpt.com/backend-api/codex` | Responses API base URL |
| `OPENAI_CODEX_AUTH_URL` | `https://auth.openai.com` | OAuth authorization server |
| `OPENAI_CODEX_CLIENT_ID` | `app_EMoamEEZ73f0CkXaXp7hrann` | OAuth client ID |

**LLM_BACKEND value:** `openai_codex`

### 9.6 Key Differences from Other Providers

- Uses Responses API (not Chat Completions) — SSE streaming
- `cost_per_token()` returns `(0, 0)` — subscription billing
- `set_model()` returns error — model fixed at construction time
- `list_models()` returns empty — no model enumeration API
- Image attachments silently dropped with a warning log

---

## 10. Gemini OAuth Provider (Cloud Code API)

**`src/llm/gemini_oauth.rs`**

### 10.1 Architecture

The Gemini OAuth provider integrates with Google's Cloud Code API for Gemini 2.0+ models, and the legacy `generativelanguage.googleapis.com` endpoint for Gemini 1.x models.

Uses official Gemini CLI OAuth credentials (public, from `google/gemini-cli` npm package).

### 10.2 Authentication

**Browser PKCE OAuth flow:**
1. Start local HTTP server on a random port
2. Open Google OAuth consent screen with PKCE challenge
3. Wait for redirect callback (or manual URL paste)
4. Exchange code for tokens
5. Discover Cloud Code project via `loadCodeAssist` API
6. Provision new project if needed via `onboardUser` API

**Credential persistence:** Saved to `~/.gemini/oauth_creds.json` (mode `0600`). Auto-refreshed before expiry (60-second buffer).

**Token refresh:** `CredentialManager` handles:
- `get_valid_credential()` — loads from disk, refreshes if expired, falls back to interactive login
- `force_refresh()` — forced refresh on 401 (used as retry-on-auth-failure)

### 10.3 Cloud Code API vs Legacy API

| Condition | API | Endpoint |
|-----------|-----|----------|
| Gemini 2.0+, 3.x, `-preview` models | Cloud Code | `cloudcode-pa.googleapis.com/v1internal:streamGenerateContent?alt=sse` |
| Gemini 1.x models | Legacy | `generativelanguage.googleapis.com/{version}/models/{model}:generateContent` |

Cloud Code requests wrap the original request body in a `{ "model": ..., "request": ..., "project": ... }` envelope and parse SSE responses.

### 10.4 Thinking Model Support

- **Gemini 3.x:** `thinkingConfig: { "thinkingLevel": "HIGH" }`
- **Gemini 2.5.x:** `thinkingConfig: { "thinkingBudget": 8192 }`
- `includeThoughts` is NOT set — IronClaw's reasoning layer strips thinking tags, so including them would cause empty responses

Thought signature injection (`SYNTHETIC_THOUGHT_SIGNATURE`) is applied to Gemini 3.x model functionCall parts to prevent 400 errors.

### 10.5 Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GEMINI_MODEL` | -- | Required. Model identifier (e.g. `gemini-2.5-flash`) |
| `GEMINI_CREDENTIALS_PATH` | `~/.gemini/oauth_creds.json` | Path to OAuth credentials file |
| `GEMINI_TOP_P` | -- | Nucleus sampling (0.0-1.0) |
| `GEMINI_TOP_K` | -- | Top-k sampling (integer) |
| `GEMINI_SEED` | -- | Deterministic generation seed |
| `GEMINI_PRESENCE_PENALTY` | -- | Presence penalty (-2.0-2.0) |
| `GEMINI_FREQUENCY_PENALTY` | -- | Frequency penalty (-2.0-2.0) |
| `GEMINI_CACHED_CONTENT` | -- | Cached content resource name |
| `GEMINI_CLI_CUSTOM_HEADERS` | -- | Custom headers (key:value,key:value) |
| `GEMINI_SAFETY_BLOCK_NONE` | `false` | Set to `true` to inject BLOCK_NONE safety settings |

**LLM_BACKEND value:** `gemini_oauth`

### 10.6 Content Curation

`curate_contents()` filters invalid model outputs before sending to the API:
- Drops empty objects `{}`
- Drops empty non-thought text parts (but preserves functionCall parts)
- Drops model turns where all parts are invalid

This prevents API 400 errors from malformed conversation history.

---

## 11. Per-Tool Reasoning System

**`src/llm/reasoning.rs`** + **`src/llm/reasoning_models.rs`**

### 11.1 Reasoning Threading

Per-tool reasoning flows through the provider chain:

1. **Provider response** — `ToolCompletionResponse` includes `tool_calls: Vec<ToolCall>`, where each `ToolCall` has an optional `reasoning: Option<String>` field
2. **Reasoning engine** (`select_tools()`, `respond_with_tools()`) — prefers per-tool reasoning from the provider; falls back to shared response content (narrative) when provider doesn't supply per-tool rationale
3. **Surface layer** — `ToolSelection.reasoning` carries the rationale to the agent worker for logging, observability, and cost tracking

```rust
// In respond_with_tools():
let tool_calls: Vec<ToolCall> = response.tool_calls.into_iter().map(|mut tc| {
    if tc.reasoning.as_ref().is_none_or(|r| r.trim().is_empty()) {
        tc.reasoning = narrative.as_ref().filter(|n| !n.is_empty()).cloned();
    } else {
        // Clean provider-supplied per-tool reasoning
        tc.reasoning = tc.reasoning.map(|r| clean_response(&truncate_at_tool_tags(&r)));
    }
    tc
}).collect();
```

### 11.2 Reasoning Model Detection

`reasoning_models.rs` contains a heuristic for detecting models with native thinking support. These models produce chain-of-thought via `reasoning_content` fields or built-in `<think>` tags:

```rust
const NATIVE_THINKING_PATTERNS: &[&str] = &[
    "qwen3", "qwq", "deepseek-r1", "deepseek-reasoner",
    "glm-z1", "glm-4-plus", "glm-5", "nanbeige",
    "step-3.5", "minimax-m2",
];
```

When a model has native thinking, IronClaw skips `<think>/<final>` prompt injection and uses a simplified system prompt: "Respond directly with your answer."

### 11.3 XML Tool-Call Recovery with Context Filtering

Some models (GLM-4.7, Qwen3, etc.) emit tool calls as XML tags in the content field instead of using the structured `tool_calls` field. `recover_tool_calls_from_content()` recovers them:

**Supported formats:**
- `<tool_call>{"name":"x","arguments":{}}</tool_call>` (JSON)
- `<tool_call>tool_name</tool_call>` (bare name)
- `<|tool_call|>...<|/tool_call|>` (pipe-delimited)
- `<function_call>...</function_call>` (function_call variant)
- `[Called tool \`name\` with arguments: {...}]` (bracket format from `flatten_tool_messages`)

**Context-aware filtering** prevents converting code examples or quoted snippets into executable tool calls:
- Tags inside fenced code blocks (``` ```) are ignored
- Tags inside inline backtick spans are ignored
- Tags inside blockquotes (`>`) are ignored
- Tags with non-empty text before/after on the same line are ignored (not isolated)
- Only tool calls whose name matches an available tool are recovered

### 11.4 Truncation at Unclosed Tool Tags (Issue #789)

`truncate_at_tool_tags()` runs before `clean_response()` and preserves text before unclosed tool-call XML tags. Without this, `strip_xml_tag()` would discard everything from an unclosed opening tag onward, leaving an empty response:

```
Input:  "Here is my analysis.\n<tool_call>{\"name\": \"read_file\"}"
Output: "Here is my analysis.\n"
```

Properly closed tags are left intact for `clean_response()` to strip normally. Tags inside code blocks are ignored.

### 11.5 Response Cleaning Pipeline

The `clean_response()` pipeline:

1. Quick-check — bail if no reasoning/final tags present
2. Build code regions (fenced blocks + inline backticks)
3. Strip thinking tags (regex, code-aware, strict mode for unclosed)
4. If `<final>` tags present: extract only `<final>` content; else use thinking-stripped text
5. Strip pipe-delimited reasoning tags (code-aware)
6. Strip tool tags (`tool_call`, `function_call`, `tool_calls` — string matching)
7. Strip bracket-format inline tool calls
8. Collapse triple+ newlines, trim

---

## 12. Session Management

`src/llm/session.rs` implements `SessionManager` for NEAR AI OAuth session
tokens.

### 12.1 Storage and Priority

Token resolution order (highest to lowest priority):

1. `NEARAI_SESSION_TOKEN` environment variable
2. Database settings table (`nearai.session_token`)
3. Disk file at `~/.ironclaw/session.json` (mode `0o600`)

### 12.2 Renewal and Thundering Herd Prevention

The session token is held in a `RwLock<SecretString>` for concurrent read
access. When renewal is needed, a `Mutex<()>` renewal lock serializes renewal
attempts. Only the first concurrent caller acquires the lock and performs
renewal; subsequent callers wait and then use the freshly renewed token.

This prevents the thundering herd problem where many simultaneous requests
all try to renew the token independently, causing multiple redundant OAuth
round trips.

### 12.3 OAuth Providers

Supports GitHub and Google OAuth flows for `private.near.ai` authentication.
For `cloud-api.near.ai`, API keys are used directly (saved to
`~/.ironclaw/.env` on first setup).

#### Codex OAuth Flow (`src/llm/codex_auth.rs`)

When `LLM_USE_CODEX_AUTH=true` is set with `LLM_BACKEND=openai`, IronClaw activates the Codex credential injection layer. The `CodexAuthProvider` (`src/llm/codex_auth.rs`) reads tokens from `~/.codex/auth.json` and wraps the `CodexChatGptProvider` (`src/llm/codex_chatgpt.rs`), which routes to the OpenAI Responses API.

**This is NOT a new `LlmBackend` enum variant** — it wraps the existing `OpenAi` backend. (v0.19.0, PR [#693](https://github.com/nearai/ironclaw/pull/693))

---

## 13. Configuration Resolution

`src/config/llm.rs` implements a three-tier priority system for all LLM
settings.

### 13.1 Priority Order

```
Environment variables  (highest -- always override)
       |
Settings database      (per-user stored preferences)
       |
Compiled defaults      (lowest -- always available)
```

This is implemented in `LlmConfig::resolve(settings: &Settings)`.

### 13.2 Provider-Specific Environment Variables

| Variable | Backend | Purpose |
|----------|---------|---------|
| `LLM_BACKEND` | All | Select backend (nearai/openai/anthropic/ollama/openai_compatible/tinfoil/github_copilot/openai_codex/gemini_oauth/bedrock) |
| `NEARAI_SESSION_TOKEN` | NearAi | Optional env override for session token (used by session manager) |
| `NEARAI_API_KEY` | NearAi | API key; auto-selects ChatCompletions mode |
| `NEARAI_MODEL` | NearAi | Primary model name |
| `NEARAI_CHEAP_MODEL` | NearAi | Lightweight model for routing/heartbeat/evaluation |
| `LLM_CHEAP_MODEL` | All | Generic cheap model (overrides NEARAI_CHEAP_MODEL, works with any backend) |
| `NEARAI_FALLBACK_MODEL` | NearAi | Failover model |
| `NEARAI_MAX_RETRIES` | NearAi | Retry count (default: 3) |
| `CIRCUIT_BREAKER_THRESHOLD` | NearAi | Failures before open (None = disabled) |
| `CIRCUIT_BREAKER_RECOVERY_SECS` | NearAi | Recovery timeout (default: 30) |
| `RESPONSE_CACHE_ENABLED` | NearAi | Enable response cache (default: false) |
| `RESPONSE_CACHE_TTL_SECS` | NearAi | Cache TTL (default: 3600) |
| `RESPONSE_CACHE_MAX_ENTRIES` | NearAi | LRU capacity (default: 1000) |
| `LLM_FAILOVER_COOLDOWN_SECS` | NearAi | Provider cooldown (default: 300) |
| `LLM_FAILOVER_THRESHOLD` | NearAi | Failures before cooldown (default: 3) |
| `OPENAI_API_KEY` | OpenAi | Required |
| `OPENAI_MODEL` | OpenAi | Default: gpt-4o |
| `OPENAI_BASE_URL` | OpenAi | Optional proxy override |
| `ANTHROPIC_API_KEY` | Anthropic | Required |
| `ANTHROPIC_MODEL` | Anthropic | Default: claude-sonnet-4-20250514 |
| `OLLAMA_BASE_URL` | Ollama | Default: <http://localhost:11434> |
| `OLLAMA_MODEL` | Ollama | Default: llama3 |
| `LLM_BASE_URL` | OpenAiCompatible | Required (e.g. OpenRouter URL) |
| `LLM_API_KEY` | OpenAiCompatible | Optional |
| `LLM_MODEL` | OpenAiCompatible | Falls back to `selected_model` from DB |
| `LLM_EXTRA_HEADERS` | OpenAiCompatible | Comma-separated `Key:Value` pairs injected into every HTTP request |
| `TINFOIL_API_KEY` | Tinfoil | Required |
| `TINFOIL_MODEL` | Tinfoil | Default: kimi-k2-5 |
| `MINIMAX_API_KEY` | MiniMax | Required. MiniMax API key |
| `MINIMAX_MODEL` | MiniMax | Default: MiniMax-M2.7 |
| `ZAI_API_KEY` | ZAi | Required. Z.AI API key |
| `ZAI_MODEL` | ZAi | Default: glm-4-plus |
| `LLM_USE_CODEX_AUTH` | OpenAi | Bool; enables Codex CLI OAuth injection (default: false) |
| `CODEX_MODEL` | OpenAi (Codex) | Model when using Codex auth (default: o4-mini) |
| `GITHUB_COPILOT_TOKEN` | GitHub Copilot | GitHub OAuth token |
| `GITHUB_COPILOT_MODEL` | GitHub Copilot | Model identifier |
| `GITHUB_COPILOT_EXTRA_HEADERS` | GitHub Copilot | Additional headers |
| `OPENAI_CODEX_MODEL` | OpenAI Codex | Default: gpt-5.3-codex |
| `OPENAI_CODEX_API_URL` | OpenAI Codex | Default: https://chatgpt.com/backend-api/codex |
| `OPENAI_CODEX_AUTH_URL` | OpenAI Codex | Default: https://auth.openai.com |
| `OPENAI_CODEX_CLIENT_ID` | OpenAI Codex | OAuth client ID |
| `GEMINI_MODEL` | Gemini OAuth | Required. Model identifier |
| `GEMINI_CREDENTIALS_PATH` | Gemini OAuth | Default: ~/.gemini/oauth_creds.json |
| `BEDROCK_REGION` | Bedrock | AWS region (default: us-east-1) |
| `BEDROCK_MODEL` | Bedrock | Required. Model ID |
| `BEDROCK_CROSS_REGION` | Bedrock | Cross-region prefix (us/eu/apac/global) |

### 13.3 Cheap Model for Lightweight Tasks

`LLM_CHEAP_MODEL` (generic, works with any backend) names a second, lower-cost
model used by `create_cheap_llm_provider()` for operations that do not need the
primary model's full capability: heartbeat checks, intent routing, and LLM-based
job evaluation. Falls back to `NEARAI_CHEAP_MODEL` when backend is `nearai`.
When not set, it falls back to the primary model.

---

## 14. Cost Accounting and Guardrails

### 14.1 Cost Table — `src/llm/costs.rs`

`model_cost(model_id)` returns per-token pricing as `(input_rate, output_rate)`
in USD with `Decimal` precision. The function strips provider prefix from model
IDs (e.g., `openai/gpt-4o` -> `gpt-4o`) before looking up the table.

Covered models include:

- OpenAI: GPT-3.5-turbo through GPT-5.3-codex, reasoning models (o1, o3, o4-mini)
- Anthropic: claude-haiku/sonnet/opus across all major versions
- Local models: zero cost via `is_local_model()` heuristic matching prefixes
  `llama`, `mistral`, `phi`, `gemma`, `qwen`, `deepseek`, `codellama`

Unknown models fall back to `default_cost()` which uses GPT-4o pricing, ensuring
cost tracking never panics but may slightly over-estimate for cheaper models.

### 14.2 CostGuard — `src/agent/cost_guard.rs`

`CostGuard` enforces two independent spending limits for autonomous/daemon modes:

**Daily budget:** Tracks cumulative USD spend via `Mutex<DailyCost>`. The
counter resets at UTC midnight. An `AtomicBool budget_exceeded` flag provides
a fast-path check that avoids acquiring the mutex on subsequent calls once
the budget is blown.

**Hourly rate limit:** Uses a `VecDeque<Instant>` sliding window of action
timestamps. Expired entries (older than 1 hour) are drained on each check.

**80% warning threshold:** A tracing `warn!` is emitted when daily spend
reaches 80% of the limit, before the hard block at 100%.

**Usage pattern:**

```rust
// BEFORE making an LLM call:
cost_guard.check_allowed().await?;  // Blocks if limit exceeded

// AFTER the call completes:
cost_guard.record_llm_call(model, input_tokens, output_tokens).await;
```

The separation of check and record means a single LLM call slot is evaluated
before commitment, but the cost is only counted after actual token consumption.

### 14.3 Gateway Status Popover (v0.10.0)

v0.10.0 added a gateway status popover in the UI that shows real-time token usage and estimated cost per session. The popover reads from the same counters updated by `CostGuard::record_llm_call()`, so the displayed figures are always consistent with the budget enforcement logic.

### 14.4 Reasoning Cost Integration

`Reasoning` (in `src/llm/reasoning.rs`) returns `TokenUsage` with every
`respond_with_tools()` call. The `agent/worker.rs` passes this to
`CostGuard::record_llm_call()` for budget tracking and to
`Estimator::record_actual()` for EMA learning.

---

## 15. Context Window Management

### 15.1 ContextMonitor — `src/agent/context_monitor.rs`

`ContextMonitor` tracks the size of the active conversation and triggers
compaction recommendations. Token estimation uses a word-count heuristic:

```
tokens = word_count * 1.3 + 4   // 1.3 tokens/word + 4 overhead for role+structure
```

This is intentionally approximate — exact tokenization would require shipping
a tokenizer per model. The conservative 1.3 multiplier slightly over-estimates,
which errs on the side of triggering compaction earlier.

**Default limits:**

- `context_limit`: 100,000 tokens
- `compaction_threshold`: 80% of limit (80,000 tokens)

**Strategy selection by fill level:**

| Fill % | Strategy |
|--------|---------|
| 80-85% | `MoveToWorkspace` (archive + keep 10 recent turns) |
| 85-95% | `Summarize` (LLM summary + keep 5 recent turns) |
| >95% | `Truncate` (drop oldest + keep 3 recent turns — no LLM call needed) |

`ContextBreakdown::analyze()` provides a per-role token breakdown
(system/user/assistant/tool) for debugging and monitoring.

### 15.2 ContextCompactor — `src/agent/compaction.rs`

`ContextCompactor` executes the strategy recommended by `ContextMonitor`.
It holds `Arc<dyn LlmProvider>` for the `Summarize` strategy.

**Summarize strategy:**

1. Collects turns to remove (all except the `keep_recent` most recent)
2. Calls the LLM with temperature 0.3, max 1024 tokens, requesting a bullet-
   point summary of key decisions, actions, and outcomes
3. Appends the summary to the workspace daily log at `daily/YYYY-MM-DD.md`
4. Truncates the thread via `thread.truncate_turns(keep_recent)`

**MoveToWorkspace strategy:**

1. Formats turns as structured markdown with turn numbers, user input, agent
   response, and tool names used
2. Appends raw content to the workspace daily log (no LLM call required)
3. Truncates to 10 recent turns

**Resilience:** Both workspace-writing strategies treat write failures as
non-fatal: a `tracing::warn!` is emitted and truncation still proceeds.
The agent never hangs on a failing workspace write.

**CompactionResult** reports `tokens_before`, `tokens_after`,
`turns_removed`, `summary_written`, and the generated `summary` text for
logging and observability.

---

## 16. Estimation, Evaluation, and Observability

### 16.1 Cost and Time Estimation — `src/estimation/`

`Estimator` combines three static estimators with an adaptive learner.

**Static estimates (per-tool lookup tables):**

| Tool category | Cost | Time |
|---------------|------|------|
| `http` | $0.0001 | 500ms |
| `echo`, `time`, `json` | $0.0000 | 10ms |
| LLM call | $0.01/1K tokens | ~50 tokens/second |

**`EstimationLearner` (EMA-based adaptive learning):**

- Tracks per-category `cost_factor` and `time_factor` (ratio of actual to
  estimated)
- Updates with Exponential Moving Average: `alpha = 0.1`
- Requires minimum 5 samples before adjusting estimates (cold-start guard)
- Confidence score: `0.5 + sample_factor * 0.3 + error_factor * 0.2`
  where `sample_factor` grows with sample count and `error_factor` decreases
  with prediction error
- Categories must be registered before use; unknown categories do not update

**`ValueEstimator`:**

- Target margin: 30% above cost
- Minimum acceptable margin: 10% above cost
- `is_profitable(cost, revenue)` -> `revenue >= cost * 1.10`
- `calculate_margin(cost, revenue)` -> `(revenue - cost) / revenue * 100`

### 16.2 Success Evaluation — `src/evaluation/`

`SuccessEvaluator` trait has two implementations:

**`RuleBasedEvaluator`:**

- Success rate threshold: 80% of tool actions must succeed
- Maximum tolerated failures: 3
- Critical error keywords: scans job output for patterns indicating hard failure
- Job state: `Completed` and `Submitted` states count as successful

**`LlmEvaluator`:**

- Sends a structured JSON prompt to the cheap LLM requesting a JSON response
  with fields `success: bool`, `confidence: f64`, and `reasoning: String`
- Falls back to `RuleBasedEvaluator` if JSON parsing fails

**`MetricsCollector`:**

- Per-tool `ToolMetrics`: call count, success/failure counts, average
  execution time, total cost
- Error categorization: timeout, rate_limit, auth, not_found, invalid_input,
  network, unknown
- `QualityMetrics`: overall success rate, average response time, total cost

### 16.3 Observability — `src/observability/`

The `Observer` trait provides a pluggable backend for event and metric recording:

```rust
pub trait Observer: Send + Sync {
    fn record_event(&self, event: ObserverEvent);
    fn record_metric(&self, metric: ObserverMetric);
    fn flush(&self);
    fn name(&self) -> &str;
}
```

**Events:** `AgentStart`, `LlmRequest`, `LlmResponse`, `ToolCallStart`,
`ToolCallEnd`, `TurnComplete`, `ChannelMessage`, `HeartbeatTick`,
`AgentEnd`, `Error`.

**Metrics:** `RequestLatency(Duration)`, `TokensUsed(u64)`,
`ActiveJobs(u64)`, `QueueDepth(u64)`.

**Backends:**

| Backend | Implementation | Overhead |
|---------|---------------|---------|
| `NoopObserver` | Empty inline methods | Zero |
| `LogObserver` | `tracing::info!` / `tracing::debug!` | Minimal |
| `MultiObserver` | Fans out to `Vec<Box<dyn Observer>>` | Per-backend |

`create_observer(config)` builds the right backend: `"log"` -> `LogObserver`,
anything else (including `"none"`, `"noop"`, unknown strings) -> `NoopObserver`.
The default backend is `"none"`.

Future backends (OpenTelemetry, Prometheus) can be added by implementing the
`Observer` trait and extending `create_observer()`.

---

## 17. Extension Points

### 17.1 Adding a New LLM Backend

1. Add a provider config struct or use `RegistryProviderConfig`
2. Implement `LlmProvider` in `src/llm/my_provider.rs`
3. For registry-based providers: add a `ProviderDefinition` to the registry
4. For special providers: add dispatch logic in `create_llm_provider()` in `src/llm/mod.rs`
5. Add env vars to `src/config/llm.rs`

The new provider immediately inherits the full reliability wrapper chain
(retry, circuit breaker, cache, failover) without any additional code.

### 17.2 Adding a New Observability Backend

1. Create `src/observability/my_backend.rs`
2. Implement `Observer` (four methods)
3. Add the match arm in `create_observer()` in `src/observability/mod.rs`
4. Expose via `pub use` in `src/observability/mod.rs`

### 17.3 Extending the Cost Table

Add entries to the match arm in `costs::model_cost()` in `src/llm/costs.rs`.
The function signature accepts any `&str` model ID and returns
`Option<(Decimal, Decimal)>`. Entries that return `None` fall through to
`default_cost()` (GPT-4o pricing).

For local models, extend the prefix list in `is_local_model()` — these return
zero cost without a table lookup.

### 17.4 Adding a Compaction Strategy

1. Add a variant to `CompactionStrategy` in `src/agent/context_monitor.rs`
2. Update `suggest_compaction()` fill-level thresholds as needed
3. Add the match arm in `ContextCompactor::compact()` in `src/agent/compaction.rs`
4. Implement the strategy method returning `CompactionPartial`

### 17.5 Custom Success Evaluation

Implement `SuccessEvaluator` in `src/evaluation/success.rs` and wire it into
the agent worker. The trait requires a single async method:

```rust
async fn evaluate(
    &self,
    job: &JobContext,
    actions: &[ActionRecord],
    output: &str,
) -> EvaluationResult;
```

`EvaluationResult` contains `success: bool`, `confidence: f64`, and
`reasoning: String`.

---

*Generated from IronClaw v0.23.0 source -- `src/llm/`, `src/config/llm.rs`,
`src/agent/cost_guard.rs`, `src/agent/context_monitor.rs`,
`src/agent/compaction.rs`, `src/estimation/`, `src/evaluation/`,
`src/observability/`.*
