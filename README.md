# IronClaw Documentation

> Comprehensive developer reference for [IronClaw](https://github.com/nearai/ironclaw) v0.7.0
> — a secure, self-hosted personal AI assistant written in Rust.

## Contents

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | Master architecture: modules, data flows, diagrams |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Build, install, configure, run as macOS/Linux service |
| [analysis/agent.md](analysis/agent.md) | Agent loop, sessions, jobs, routines, heartbeat |
| [analysis/channels.md](analysis/channels.md) | REPL, web gateway, HTTP, WASM, webhook channels |
| [analysis/cli.md](analysis/cli.md) | CLI subcommands, doctor, service manager, registry |
| [analysis/config.md](analysis/config.md) | Configuration system, env vars, all config structs |
| [analysis/db-storage.md](analysis/db-storage.md) | Database backends (libsql/postgres), migrations, schema |
| [analysis/llm.md](analysis/llm.md) | LLM backends, multi-provider, retry, cost guard |
| [analysis/safety-sandbox.md](analysis/safety-sandbox.md) | Safety layer, WASM sandbox, Docker orchestrator, proxy |
| [analysis/tools.md](analysis/tools.md) | Tool system, built-in tools, MCP client, WASM tools |
| [analysis/skills-extensions.md](analysis/skills-extensions.md) | Skills system, WASM channels, extensions, hooks |
| [analysis/workspace-memory.md](analysis/workspace-memory.md) | Workspace FS, semantic memory, embeddings, search |
| [analysis/secrets-keychain.md](analysis/secrets-keychain.md) | Secrets store, keychain, crypto, credential injection |
| [analysis/tunnels-pairing.md](analysis/tunnels-pairing.md) | Tunnels (cloudflare/ngrok/tailscale), mobile pairing |
| [analysis/worker-orchestrator.md](analysis/worker-orchestrator.md) | Worker runtime, Claude bridge, proxy LLM, Docker sandbox |

## About IronClaw

IronClaw is a Rust-based personal AI assistant with:
- **Multi-channel support**: REPL, web gateway, HTTP webhooks, WASM plugin channels
- **Security-first**: WASM sandbox, Docker isolation, credential protection, SSRF proxy
- **Self-expanding**: Dynamic WASM tool building, MCP protocol, plugin architecture
- **Persistent memory**: Hybrid FTS+vector search, workspace filesystem, identity files
- **Multiple LLM backends**: Anthropic, OpenAI, Gemini, OpenAI-compatible (any endpoint)

## Version

Documented: IronClaw v0.7.0
Source: [github.com/nearai/ironclaw](https://github.com/nearai/ironclaw)
