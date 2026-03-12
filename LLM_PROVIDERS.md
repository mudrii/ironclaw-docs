# LLM Provider Configuration

> Version baseline: IronClaw v0.18.0 (`v0.18.0` tag snapshot)

IronClaw defaults to NEAR AI for model access, but supports any OpenAI-compatible
endpoint as well as Anthropic and Ollama directly. This guide covers the most common
configurations.

## Provider Overview

| Provider | Backend value | Requires API key | Notes |
|---|---|---|---|
| NEAR AI | `nearai` | Optional (`NEARAI_API_KEY`) | Default; OAuth/session auth by default, API-key mode also supported |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` | Claude models |
| OpenAI | `openai` | `OPENAI_API_KEY` | GPT models |
| Ollama | `ollama` | No | Local inference |
| OpenRouter | `openai_compatible` | `LLM_API_KEY` | 200+ models (wizard text; platform catalog varies); dedicated wizard preset (v0.12.0) |
| Together AI | `openai_compatible` | `LLM_API_KEY` | Fast inference |
| Fireworks AI | `openai_compatible` | `LLM_API_KEY` | Fast inference |
| vLLM / LiteLLM | `openai_compatible` | Optional | Self-hosted |
| LM Studio | `openai_compatible` | No | Local GUI |
| Tinfoil | `tinfoil` | `TINFOIL_API_KEY` | Private TEE inference |
| Google Gemini | `openai_compatible` | `LLM_API_KEY` | Via OpenAI-compatible endpoint; or use `gemini` backend (v0.17.0) |
| AWS Bedrock | `bedrock` | IAM/SSO | Native Converse API; requires `--features bedrock` (v0.17.0) |
| Mistral AI | `openai_compatible` | `LLM_API_KEY` | Direct Mistral endpoint (v0.17.0) |
| io.net | `openai_compatible` | `LLM_API_KEY` | Decentralized GPU cloud (v0.17.0) |
| Yandex YandexGPT | `openai_compatible` | `LLM_API_KEY` | Yandex Cloud AI (v0.17.0) |
| Cloudflare WS AI | `openai_compatible` | `LLM_API_KEY` | Cloudflare Workers AI gateway (v0.17.0) |

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

### Anthropic OAuth Onboarding (v0.17.0)

As of v0.17.0, `ironclaw onboard` supports Anthropic OAuth login using a setup token:

```bash
ironclaw onboard --provider-only
# Select "Anthropic" — the wizard opens a browser for OAuth
```

Standard API keys (`sk-ant-...`) are still fully supported and preferred for non-interactive/service deployments.

### Anthropic Prompt Caching (v0.17.0)

IronClaw automatically injects `cache_control` breakpoints for Anthropic models on eligible
long-context messages, reducing costs for repeated context (e.g., workspace memory, long
system prompts). No configuration required — enabled automatically when `LLM_BACKEND=anthropic`.

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
| `TINFOIL_API_KEY` | — | Required. Tinfoil API key |
| `TINFOIL_MODEL` | `kimi-k2-5` | Model identifier |

---

## Google Gemini (v0.17.0)

Google Gemini models are available via the OpenAI-compatible endpoint:

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
LLM_API_KEY=your-gemini-api-key
LLM_MODEL=gemini-2.0-flash
```

Popular Gemini model IDs:

| Model | ID |
|---|---|
| Gemini 2.0 Flash | `gemini-2.0-flash` |
| Gemini 2.0 Flash-Lite | `gemini-2.0-flash-lite` |
| Gemini 1.5 Pro | `gemini-1.5-pro` |

---

## AWS Bedrock (v0.17.0)

AWS Bedrock requires building IronClaw with the `bedrock` feature flag:

```bash
cargo build --release --features bedrock
```

Then configure:

```env
LLM_BACKEND=bedrock
BEDROCK_REGION=us-east-1
BEDROCK_MODEL=anthropic.claude-sonnet-4-5-20251001-v2:0
# Uses AWS credential chain (IAM role, environment, ~/.aws/credentials, SSO)
```

For AWS SSO profiles:

```env
AWS_PROFILE=my-sso-profile
```

Popular Bedrock model IDs:

| Model | ID |
|---|---|
| Claude Sonnet 4.5 | `anthropic.claude-sonnet-4-5-20251001-v2:0` |
| Claude Haiku 3.5 | `anthropic.claude-haiku-3-5-20241022-v1:0` |
| Llama 3.3 70B | `meta.llama3-3-70b-instruct-v1:0` |
| Amazon Nova Pro | `amazon.nova-pro-v1:0` |

---

## Mistral AI (v0.17.0)

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://api.mistral.ai/v1
LLM_API_KEY=your-mistral-api-key
LLM_MODEL=mistral-small-latest
```

---

## io.net (v0.17.0)

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://api.io.net/v1
LLM_API_KEY=your-ionet-api-key
LLM_MODEL=meta-llama/Llama-3.3-70B-Instruct
```

---

## Cloudflare Workers AI (v0.17.0)

```env
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://api.cloudflare.com/client/v4/accounts/YOUR_ACCOUNT_ID/ai/v1
LLM_API_KEY=your-cloudflare-api-token
LLM_MODEL=@cf/meta/llama-3.3-70b-instruct-fp8-fast
```

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
LLM_BACKEND=nearai                         # Smart routing applies to NearAI backend
NEARAI_MODEL=zai-org/GLM-latest           # Primary (capable) model — used for Pro/Frontier tiers
NEARAI_CHEAP_MODEL=zai-org/GLM-flash      # Cheap model — used for Flash/Standard tiers
SMART_ROUTING_CASCADE=true                # Retry with primary if cheap model gives uncertain Pro-tier response
```

**Four tiers (score 0–100):**

| Score | Tier | Routed to |
|-------|------|-----------|
| 0–15 | Flash | Cheap model |
| 16–40 | Standard | Cheap model |
| 41–65 | Pro | Cheap model (escalates to primary if SMART_ROUTING_CASCADE=true and response is uncertain) |
| 66–100 | Frontier | Primary model always |

**Pattern overrides** (bypass scoring, applied before the scorer):
- Greetings and short yes/no questions → **Flash** (fast-path)
- Security audits, CVE analysis, cryptography questions → **Frontier** (always primary)

**13 scoring dimensions** (partial list — see `src/llm/smart_routing.rs`):
technical depth, code complexity, reasoning chains, context length, ambiguity, domain expertise required, multi-step planning, creativity, factual precision, adversarial robustness, mathematical complexity, structured output requirements, time-sensitivity.

**Cascade mode** (`SMART_ROUTING_CASCADE=true`): applies only to **Pro-tier** prompts routed to the cheap model. If the cheap model's response shows uncertainty signals (hedging phrases, incomplete reasoning), the request is automatically re-sent to the primary model.

---

## LLM Request Timeout (v0.17.0)

Configure how long IronClaw waits for LLM responses before timing out:

```env
LLM_REQUEST_TIMEOUT_SECS=120    # Default: 120 seconds
```

Increase this for slow local models (Ollama) or large-context requests.

---
