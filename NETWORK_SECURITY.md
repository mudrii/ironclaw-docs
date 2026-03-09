# IronClaw Network Security Reference

> Version baseline: IronClaw v0.16.1 (`v0.16.1` tag snapshot)
> Validated against source modules in `src/channels/`, `src/orchestrator/`, `src/sandbox/`, `src/tools/builtin/http.rs`, and `src/channels/wasm/signature.rs`

This document summarizes the network-facing surfaces and egress controls in the released `v0.16.1` codebase. It complements the deeper module analyses in:

- [analysis/channels.md](analysis/channels.md)
- [analysis/safety-sandbox.md](analysis/safety-sandbox.md)
- [analysis/tools.md](analysis/tools.md)
- [analysis/secrets-keychain.md](analysis/secrets-keychain.md)

## 1. Inbound Network Surfaces

| Surface | Default bind | Auth | Notes |
|---|---|---|---|
| Web gateway | `127.0.0.1:3000` | Bearer token | Browser UI + JSON API + SSE + WS |
| HTTP webhook server | `0.0.0.0:8080` | Shared secret | Hosts built-in HTTP webhook routes and WASM webhook fragments |
| Orchestrator internal API | `127.0.0.1:50051` on macOS/Windows, `0.0.0.0:50051` on Linux | Per-job bearer token | Worker/container control plane |
| OAuth callback route (gateway) | same as gateway | none on callback endpoint itself | Used for hosted/remote OAuth callback flow |
| Local OAuth listener | loopback `127.0.0.1:9876` by default | none | Short-lived CLI/local OAuth callback server |
| Sandbox proxy | loopback ephemeral port | loopback-only | Egress control point for sandbox containers |

## 2. Web Gateway Security

Source: `src/channels/web/auth.rs`, `src/channels/web/server.rs`, `src/channels/web/sse.rs`

### Authentication

The gateway uses a bearer token checked with constant-time comparison (`subtle::ConstantTimeEq`).

Accepted auth paths:
- `Authorization: Bearer <token>` header
- `?token=<token>` query parameter **only** on streaming GET endpoints:
  - `/api/chat/events`
  - `/api/logs/events`
  - `/api/chat/ws`

This `v0.15.0` restriction prevents query-token use on state-changing or general API routes.

### Public routes

Public by default:
- `/api/health`
- `/oauth/callback`
- embedded static assets (`/`, `/app.js`, `/style.css`, favicon)

Protected routes include chat, memory, jobs, logs, extensions, pairing, routines, settings, skills, and OpenAI-compatible API endpoints.

### Additional controls

- CORS allowlist is limited to local gateway origins
- WebSocket route validates `Origin` in addition to bearer auth
- chat send path uses a sliding-window rate limiter (`30` requests / `60` seconds)
- request body limit is `1 MiB`
- SSE + WebSocket connection count is capped at `100` combined
- security headers include `X-Content-Type-Options: nosniff` and `X-Frame-Options: DENY`

## 3. HTTP Webhook Security

Source: `src/channels/http.rs`, `src/channels/webhook_server.rs`, `src/channels/wasm/signature.rs`

### Built-in HTTP webhook channel

The built-in HTTP channel:
- expects JSON input
- validates a shared secret from the request body using constant-time comparison
- applies request-size and rate-limit controls

### WASM channel webhook security

WASM channel routes are hosted behind the unified webhook server and can enforce additional signature verification:

- **Discord-style Ed25519 signatures** via `X-Signature-Ed25519` + timestamp
- **Slack-style HMAC-SHA256 signatures** via `X-Slack-Signature` + `X-Slack-Request-Timestamp`

For Slack-style verification in `v0.16.0+`:
- the runtime computes `v0:{timestamp}:{body}`
- verifies HMAC-SHA256 using the configured signing secret
- uses constant-time comparison
- enforces a 5-minute replay window

## 4. Orchestrator Internal API

Source: `src/orchestrator/api.rs`, `src/orchestrator/auth.rs`

The orchestrator is the control plane used by sandbox workers.

Security properties:
- every `/worker/{job_id}/...` route is protected by middleware
- auth tokens are per-job, randomly generated, and validated with constant-time comparison
- tokens are scoped to a specific job ID
- credential grants are per-job and revoked with the token
- `/health` is the only unauthenticated endpoint

Operational caveat:
- on Linux, the orchestrator binds `0.0.0.0` because Docker bridge networking cannot reach host loopback directly; firewalling remains important even though worker routes require tokens

## 5. OAuth Callback Paths

Source: `src/cli/oauth_defaults.rs`, `src/channels/web/server.rs`

IronClaw supports two OAuth callback modes:

1. **Local loopback listener** for local/CLI flows
2. **Gateway callback route** (`/oauth/callback`) for hosted or remote-server deployments

This matches the `v0.15.0` change routing provider callbacks through the web gateway for hosted instances.

## 6. Egress Controls

## 6.1 Built-in `http` tool

Source: `src/tools/builtin/http.rs`

The built-in `http` tool applies explicit SSRF and exfiltration defenses:

- HTTPS only
- blocks `localhost`, `*.localhost`, loopback, private IPs, link-local, multicast, unspecified addresses, and `169.254.169.254`
- resolves DNS and rejects hostnames that map to disallowed IPs (DNS rebinding defense)
- max response size `5 MiB`
- leak detection scans outbound URL/headers/body before sending
- simple GET requests without auth/headers/body require no approval and may follow up to `3` redirects, with SSRF validation on every hop
- non-simple requests do **not** follow redirects and require approval unless auto-approved

## 6.2 WASM tool and channel HTTP

Source: `src/tools/wasm/*`, `src/channels/wasm/*`, `src/safety/leak_detector.rs`

WASM execution uses host-side controls:
- allowlisted outbound endpoints from capabilities metadata
- credential injection at the host boundary
- leak detection on requests/responses
- runtime resource limits
- WIT/capability validation before activation

Secrets are injected by host policy; the WASM guest does not read raw secrets directly.

## 6.3 Sandbox proxy

Source: `src/sandbox/proxy/*`

For Docker sandbox execution, outbound HTTP is forced through a loopback proxy with:
- domain allowlisting
- HTTP credential injection when allowed
- CONNECT tunnel handling for HTTPS
- hop-by-hop header stripping
- shutdown signaling and bounded proxy lifecycle

## 7. Secret and Credential Protections

Source: `src/secrets/*`, `src/safety/*`

At `v0.16.1`, security-critical randomness uses `OsRng` rather than `thread_rng()`.

Credential protections include:
- encrypted at-rest secret storage
- per-secret HKDF-derived encryption keys
- host-side credential injection
- leak detection on both request and response paths
- no LLM exposure of raw secret values in normal tool flow

## 8. Main Operational Caveats

These are expected deployment caveats, not undocumented surprises:

- no app-layer TLS termination is built into the gateway; use a reverse proxy or tunnel for remote exposure
- the webhook server defaults to `0.0.0.0` for webhook reachability
- the Linux orchestrator listener also binds `0.0.0.0`
- the orchestrator API does not currently add its own request-rate limiter
- orchestrator shutdown is not as graceful as the main gateway/webhook listeners

## 9. Release-Specific Security Changes Covered Here

### v0.15.0
- query-token gateway auth restricted to SSE/streaming GET endpoints only
- OAuth callbacks can route through the gateway for hosted deployments

### v0.16.0
- Slack HMAC-SHA256 webhook signature validation added
- security-critical randomness migrated to `OsRng`
- unified `http` tool approval model documented by actual source behavior

### v0.16.1
- no major new network surface; release mainly reverted premature WASM registry SHA256 values to `null`

## 10. Validation Sources

Primary source files reviewed for this document:
- `src/channels/web/auth.rs`
- `src/channels/web/server.rs`
- `src/channels/web/sse.rs`
- `src/channels/http.rs`
- `src/channels/webhook_server.rs`
- `src/channels/wasm/signature.rs`
- `src/orchestrator/api.rs`
- `src/orchestrator/auth.rs`
- `src/cli/oauth_defaults.rs`
- `src/tools/builtin/http.rs`
- `src/sandbox/proxy/http.rs`
- `src/sandbox/proxy/allowlist.rs`
- `src/secrets/crypto.rs`
