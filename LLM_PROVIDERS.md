# LLM Provider Configuration

> Version baseline: IronClaw v0.25.0 (`v0.25.0` tag snapshot)

IronClaw defaults to NEAR AI for model access, but supports any OpenAI-compatible
endpoint as well as Anthropic, Ollama, GitHub Copilot, OpenAI Codex, Gemini, and AWS Bedrock directly. This guide covers the most common configurations.

## Provider Overview

| Provider | Backend value | Requires API key | Notes |
|---|---|---|---|
| NEAR AI | `nearai` | Optional (`NEARAI_API_KEY`) | Default; OAuth/session auth by default, API-key mode also supported |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` | Claude models |
| OpenAI | `openai` | `OPENAI_API_KEY` | GPT models |
| Ollama | `ollama` | No | Local inference |
| OpenRouter | `openai_compatible` | `LLM_API_KEY` | 200+ models; dedicated wizard preset (v0.12.0) |
| Together AI | `openai_compatible` | `LLM_API_KEY` | Fast inference |
| Fireworks AI | `openai_compatible` | `LLM_API_KEY` | Fast inference |
| vLLM / LiteLLM | `openai_compatible` | Optional | Self-hosted |
| LM Studio | `openai_compatible` | No | Local GUI |
| Tinfoil | `tinfoil` | `TINFOIL_API_KEY` | Private TEE inference |
| MiniMax | `minimax` | `MINIMAX_API_KEY` | MiniMax-M2.7 models |
| Z.AI | `zai` | `ZAI_API_KEY` | GLM-5 series models from Z.AI/Zhipu |
| Codex / ChatGPT | `openai` + `LLM_USE_CODEX_AUTH=true` | (Codex CLI OAuth token) | OpenAI ChatGPT via Codex OAuth -- no separate API key |
| GitHub Copilot | `github_copilot` | `GITHUB_COPILOT_TOKEN` | GitHub Copilot Chat API with device-login OAuth |
| OpenAI Codex | `openai_codex` | (subscription OAuth) | ChatGPT subscription via Responses API; requires `ironclaw login --openai-codex` |
| Gemini | `gemini_oauth` | (Google OAuth) | Gemini models via Cloud Code API; free-tier subscription |
| AWS Bedrock | `bedrock` | IAM / SSO | Native Converse API (requires `--features bedrock` at build time) |

**Total: 17+ provider configurations** (including OpenAI-compatible endpoints).

---

## NEAR AI (default)

No additional configuration required. On first run, `ironclaw onboard` opens a browser
for OAuth authentication. Session credentials are saved to `NEARAI_SESSION_PATH` (defaults
to `~/.ironclaw/session.json`, resolved via base-dir helpers).

```env
NEARAI_MODEL=zai-org/GLM-latest        # Default if unset: zai-org/GLM-latest
# Override example: NEARAI_MODEL=claude-3-5-sonnet-20241022
NEARAI_BASE_URL=https://private.near.ai
```

---

## Anthropic (Claude)

```env
LLM_BACKEND=anthropic
ANTHROPIC_API_KEY=sk-ant-...
```

Popular models: `claude-sonnet-4-20250514`, `claude-3-5-sonnet-20241022`, `claude-3-5-haiku-20241022`

---

## OpenAI (GPT)

```env
LLM_BACKEND=openai
OPENAI_API_KEY=sk-...
```

Popular models: `gpt-4o`, `gpt-4o-mini`, `o3-mini`

---

## Ollama (local)

Install Ollama from [ollama.com](https://ollama.com), pull a model, then:

```env
LLM_BACKEND=ollama
OLLAMA_MODEL=llama3.2                  # Default if unset: llama3
# OLLAMA_BASE_URL=http://localhost:11434   # default
```

Pull a model first: `ollama pull llama3.2`

### Ollama as Embeddings Provider

Ollama can also serve as a local embeddings provider for fully offline operation:

```env
EMBEDDING_PROVIDER=ollama
EMBEDDING_MODEL=nomic-embed-text   # or mxbai-embed-large, all-minilm
OLLAMA_BASE_URL=http://localhost:11434
EMBEDDING_ENABLED=true
```

---

## OpenAI-Compatible Endpoints

All providers below use `LLM_BACKEND=openai_compatible`. Set `LLM_BASE_URL` to the
provider's OpenAI-compatible endpoint and `LLM_API_KEY` to your API key.

### OpenRouter

[OpenRouter](https://openrouter.ai) routes to a large cross-provider model catalog from a single API key.

As of v0.12.0, the setup wizard includes **OpenRouter** as a dedicated preset option (not just 'OpenAI-compatible'). Select **OpenRouter** during `ironclaw onboard` to have the base URL automatically configured as `https://openrouter.ai/api/v1`.

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_API_KEY=sk-or-...
LLM_MODEL=anthropic/claude-sonnet-4
# Optional: Attribution headers recommended by OpenRouter
LLM_EXTRA_HEADERS=HTTP-Referer:https://myapp.com,X-Title:MyApp
```

`LLM_EXTRA_HEADERS` accepts a comma-separated list of `Key:Value` pairs injected into every LLM request. Useful for OpenRouter attribution headers or provider-specific requirements.

Popular OpenRouter model IDs:

| Model | ID |
|---|---|
| Claude Sonnet 4 | `anthropic/claude-sonnet-4` |
| GPT-4o | `openai/gpt-4o` |
| Llama 4 Maverick | `meta-llama/llama-4-maverick` |
| Gemini 2.0 Flash | `google/gemini-2.0-flash-001` |
| Mistral Small | `mistralai/mistral-small-3.1-24b-instruct` |

Browse all models at [openrouter.ai/models](https://openrouter.ai/models).

### Together AI

[Together AI](https://www.together.ai) provides fast inference for open-source models.

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://api.together.xyz/v1
LLM_API_KEY=...
LLM_MODEL=meta-llama/Llama-3.3-70B-Instruct-Turbo
```

Popular Together AI model IDs:

| Model | ID |
|---|---|
| Llama 3.3 70B | `meta-llama/Llama-3.3-70B-Instruct-Turbo` |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| Qwen 2.5 72B | `Qwen/Qwen2.5-72B-Instruct-Turbo` |

### Fireworks AI

[Fireworks AI](https://fireworks.ai) offers fast inference with compound AI system support.

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://api.fireworks.ai/inference/v1
LLM_API_KEY=fw_...
LLM_MODEL=accounts/fireworks/models/llama4-maverick-instruct-basic
```

### vLLM / LiteLLM (self-hosted)

For self-hosted inference servers:

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=http://localhost:8000/v1
LLM_API_KEY=token-abc123        # set to any string if auth is not configured
LLM_MODEL=meta-llama/Llama-3.1-8B-Instruct
```

LiteLLM proxy (forwards to any backend, including Bedrock, Vertex, Azure):

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=http://localhost:4000/v1
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4o                 # as configured in litellm config.yaml
```

### LM Studio (local GUI)

Start LM Studio's local server, then:

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=http://localhost:1234/v1
LLM_MODEL=llama-3.2-3b-instruct-q4_K_M
# LLM_API_KEY is not required for LM Studio
```

---

## Tinfoil (Private TEE Inference)

Tinfoil runs models inside hardware-attested Trusted Execution Environments (TEEs), ensuring your prompts and completions are private even from the server operator.

```env
LLM_BACKEND=tinfoil
TINFOIL_API_KEY=your-tinfoil-api-key
TINFOIL_MODEL=kimi-k2-5          # Default model
```

| Variable | Default | Description |
|----------|---------|-------------|
| `TINFOIL_API_KEY` | -- | Required. Tinfoil API key |
| `TINFOIL_MODEL` | `kimi-k2-5` | Model identifier |

---

## MiniMax

MiniMax is a built-in LLM provider. Default model updated to MiniMax-M2.7 in v0.23.0.

```env
LLM_BACKEND=minimax
MINIMAX_API_KEY=your-minimax-api-key
# MINIMAX_MODEL=MiniMax-M2.7   # Default model
```

| Variable | Default | Description |
|----------|---------|-------------|
| `MINIMAX_API_KEY` | -- | Required. MiniMax API key |
| `MINIMAX_MODEL` | `MiniMax-M2.7` | Model identifier (`MiniMax-M2.7`, `MiniMax-M2.7-highspeed`) |

---

## Z.AI (GLM-5)

Z.AI provides access to GLM-5 series models. Built-in provider (v0.19.0, [#938](https://github.com/nearai/ironclaw/pull/938)).

```env
LLM_BACKEND=zai
ZAI_API_KEY=your-zai-api-key
# ZAI_MODEL=glm-4-plus   # Default model
```

| Variable | Default | Description |
|----------|---------|-------------|
| `ZAI_API_KEY` | -- | Required. Z.AI API key |
| `ZAI_MODEL` | `glm-4-plus` | Model identifier |

---

## Codex / ChatGPT (via Codex CLI OAuth)

IronClaw can reuse existing Codex CLI OAuth tokens to call the ChatGPT backend via OpenAI's Responses API. No separate API key is needed if Codex CLI is already authenticated (v0.19.0, [#693](https://github.com/nearai/ironclaw/pull/693)).

This is **not** a new `LLM_BACKEND` value -- it is a credential injection layer on top of `openai`. Activated by `LLM_USE_CODEX_AUTH=true`.

```env
LLM_BACKEND=openai
LLM_USE_CODEX_AUTH=true
# CODEX_MODEL=o4-mini    # Default model
# No API key needed -- reads from ~/.codex/auth.json
```

**Prerequisites:** Codex CLI must be installed and authenticated (`codex auth`).

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_USE_CODEX_AUTH` | `false` | Set to `true` to use Codex CLI OAuth credentials |
| `CODEX_MODEL` | `o4-mini` | Model identifier |

---

## GitHub Copilot

GitHub Copilot provider with direct HTTP and automatic token exchange. Uses the Copilot Chat API at `api.githubcopilot.com` which speaks OpenAI Chat Completions format.

**Authentication:** Two-step flow -- a long-lived GitHub OAuth token is exchanged for a short-lived Copilot session token. IronClaw handles this automatically via `CopilotTokenManager`.

**Setup:** Run `ironclaw onboard` and select GitHub Copilot, or provide a token directly:

```env
LLM_BACKEND=github_copilot
GITHUB_COPILOT_TOKEN=gho_xxxxxxxxxxxx
# GITHUB_COPILOT_MODEL=gpt-4o   # Model identifier
```

The setup wizard supports GitHub device login (browser-based OAuth) or manual token paste from your IDE's Copilot sign-in (`~/.config/github-copilot/apps.json`).

| Variable | Default | Description |
|----------|---------|-------------|
| `GITHUB_COPILOT_TOKEN` | -- | Required. GitHub OAuth token (from device login or IDE) |
| `GITHUB_COPILOT_MODEL` | -- | Model identifier |
| `GITHUB_COPILOT_EXTRA_HEADERS` | -- | Additional headers injected into requests |

**Requirements:** An active GitHub Copilot subscription (Individual, Business, or Enterprise).

**Known limitation:** The device login flow uses VS Code Copilot's OAuth client ID and editor identity headers. GitHub could rotate this client ID at any time. If GitHub publishes an official third-party client ID, it should be adopted immediately.

---

## OpenAI Codex (ChatGPT Subscription)

OpenAI Codex uses the Responses API at `chatgpt.com/backend-api/codex/responses` with ChatGPT subscription OAuth tokens. This is **subscription-based billing** -- no per-token API costs.

**Setup:** Run the device code login flow:

```bash
ironclaw login --openai-codex
```

This displays a code for you to enter at `auth.openai.com/codex/device`. Tokens are persisted to `~/.ironclaw/openai_codex_session.json` and auto-refreshed before expiry.

```env
LLM_BACKEND=openai_codex
# OPENAI_CODEX_MODEL=gpt-5.3-codex      # Default model
# OPENAI_CODEX_API_URL=https://chatgpt.com/backend-api/codex  # Default
```

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENAI_CODEX_MODEL` | `gpt-5.3-codex` | Model identifier |
| `OPENAI_CODEX_API_URL` | `https://chatgpt.com/backend-api/codex` | Responses API base URL |
| `OPENAI_CODEX_AUTH_URL` | `https://auth.openai.com` | OAuth authorization server |
| `OPENAI_CODEX_CLIENT_ID` | `app_EMoamEEZ73f0CkXaXp7hrann` | OAuth client ID |

**Requirements:** An active ChatGPT Plus, Pro, or Team subscription.

**Key differences:**
- Uses Responses API (not Chat Completions)
- Zero API cost -- billing through subscription
- Model is fixed at construction time (cannot switch at runtime)
- Image attachments are silently dropped
- Token refresh is automatic (pre-emptive refresh before expiry + retry on 401)

---

## Gemini (Google Cloud Code API)

Gemini OAuth provider integrates with Google's Cloud Code API for Gemini 2.0+ models. Uses official Gemini CLI OAuth credentials (public, from `google/gemini-cli`).

**Setup:** Run the OAuth login flow:

```bash
ironclaw login --gemini
```

This opens a browser for Google OAuth consent. Credentials are saved to `~/.gemini/oauth_creds.json`. On first login, a free-tier Cloud Code project is automatically provisioned.

```env
LLM_BACKEND=gemini_oauth
GEMINI_MODEL=gemini-2.5-flash
# GEMINI_CREDENTIALS_PATH=~/.gemini/oauth_creds.json  # Default
```

| Variable | Default | Description |
|----------|---------|-------------|
| `GEMINI_MODEL` | -- | Required. Model identifier (e.g. `gemini-2.5-flash`, `gemini-3-pro-preview`) |
| `GEMINI_CREDENTIALS_PATH` | `~/.gemini/oauth_creds.json` | Path to OAuth credentials file |
| `GEMINI_TOP_P` | -- | Nucleus sampling (0.0-1.0) |
| `GEMINI_TOP_K` | -- | Top-k sampling (integer) |
| `GEMINI_SEED` | -- | Deterministic generation seed |
| `GEMINI_PRESENCE_PENALTY` | -- | Presence penalty (-2.0-2.0) |
| `GEMINI_FREQUENCY_PENALTY` | -- | Frequency penalty (-2.0-2.0) |
| `GEMINI_CACHED_CONTENT` | -- | Cached content resource name |
| `GEMINI_CLI_CUSTOM_HEADERS` | -- | Custom headers (key:value,key:value) |
| `GEMINI_SAFETY_BLOCK_NONE` | `false` | Set to `true` for BLOCK_NONE safety settings |

**Available models:**
- `gemini-3.1-pro-preview`, `gemini-3-pro-preview` (2M context)
- `gemini-3-flash-preview`, `gemini-3.1-flash-lite-preview` (1M context)
- `gemini-2.5-pro` (2M context), `gemini-2.5-flash`, `gemini-2.5-flash-lite` (1M context)

**API routing:**
- Gemini 2.0+ models use Cloud Code API (`cloudcode-pa.googleapis.com`)
- Gemini 1.x models use legacy API (`generativelanguage.googleapis.com`)

**Thinking model support:**
- Gemini 3.x: `thinkingLevel: HIGH`
- Gemini 2.5.x: `thinkingBudget: 8192`

---

## AWS Bedrock

AWS Bedrock uses the native Converse API. Requires `--features bedrock` at build time (not in default features due to heavy AWS SDK dependencies).

```env
LLM_BACKEND=bedrock
BEDROCK_MODEL=anthropic.claude-opus-4-6-v1
# BEDROCK_REGION=us-east-1        # Default
# BEDROCK_CROSS_REGION=us         # Optional: us, eu, apac, global
```

| Variable | Default | Description |
|----------|---------|-------------|
| `BEDROCK_MODEL` | -- | Required. Model ID (e.g. `anthropic.claude-opus-4-6-v1`) |
| `BEDROCK_REGION` | `us-east-1` | AWS region |
| `BEDROCK_CROSS_REGION` | -- | Cross-region inference prefix |
| `AWS_PROFILE` | -- | AWS named profile for SSO/assume-role |

**Auth:** Standard AWS credential chain -- IAM credentials, SSO profiles, or instance roles.

---

## Using the Setup Wizard

Instead of editing `.env` manually, run the onboarding wizard:

```bash
ironclaw onboard
```

As of v0.12.0, select **OpenRouter** directly from the wizard for a one-step setup that
automatically sets the base URL to `https://openrouter.ai/api/v1`. For other providers,
select **"OpenAI-compatible"** (Together AI, Fireworks, vLLM, LiteLLM, or LM Studio).
You will be prompted for the base URL and (optionally) an API key.
The model name is configured in the following step.

---

## Smart Routing (Cost Optimization)

Smart routing (**redesigned in v0.16.0**, PR #529) uses a 13-dimension complexity scorer to classify every prompt into one of four tiers, then routes to the appropriate model.

```env
LLM_BACKEND=nearai                         # Smart routing applies to any backend
NEARAI_MODEL=zai-org/GLM-latest           # Primary (capable) model -- used for Pro/Frontier tiers
NEARAI_CHEAP_MODEL=zai-org/GLM-flash      # Cheap model -- used for Flash/Standard tiers
# Or use the generic cheap model (works with any backend):
# LLM_CHEAP_MODEL=gpt-4o-mini
SMART_ROUTING_CASCADE=true                # Retry with primary if cheap model gives uncertain Pro-tier response
```

**Four tiers (score 0-100):**

| Score | Tier | Routed to |
|-------|------|-----------|
| 0-15 | Flash | Cheap model |
| 16-40 | Standard | Cheap model |
| 41-65 | Pro | Cheap model (escalates to primary if SMART_ROUTING_CASCADE=true and response is uncertain) |
| 66-100 | Frontier | Primary model always |

**Pattern overrides** (bypass scoring, applied before the scorer):
- Greetings and short yes/no questions -> **Flash** (fast-path)
- Security audits, CVE analysis, cryptography questions -> **Frontier** (always primary)

**13 scoring dimensions** (partial list -- see `src/llm/smart_routing.rs`):
technical depth, code complexity, reasoning chains, context length, ambiguity, domain expertise required, multi-step planning, creativity, factual precision, adversarial robustness, mathematical complexity, structured output requirements, time-sensitivity.

**Cascade mode** (`SMART_ROUTING_CASCADE=true`): applies only to **Pro-tier** prompts routed to the cheap model. If the cheap model's response shows uncertainty signals (hedging phrases, incomplete reasoning), the request is automatically re-sent to the primary model.
