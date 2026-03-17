# Changelog

> Version baseline: IronClaw v0.19.0 (`v0.19.0` tag snapshot)
> Scope: mirrored release changelog entries for the latest relevant released versions

All notable changes to this project are documented upstream in `nearai/ironclaw`. This mirror intentionally focuses on the latest released versions relevant to the current docs baseline.

## [Unreleased]

## [0.19.0](https://github.com/nearai/ironclaw/compare/v0.18.0...v0.19.0) - 2026-03-17

### Added

- verify telegram owner during hot activation ([#1157](https://github.com/nearai/ironclaw/pull/1157))
- *(config)* unify config resolution with Settings fallback (Phase 2, #1119) ([#1203](https://github.com/nearai/ironclaw/pull/1203))
- *(sandbox)* add retry logic for transient container failures ([#1232](https://github.com/nearai/ironclaw/pull/1232))
- *(heartbeat)* fire_at time-of-day scheduling with IANA timezone ([#1029](https://github.com/nearai/ironclaw/pull/1029))
- Reuse Codex CLI OAuth tokens for ChatGPT backend LLM calls ([#693](https://github.com/nearai/ironclaw/pull/693))
- add pre-push git hook with delta lint mode ([#833](https://github.com/nearai/ironclaw/pull/833))
- *(cli)* add `logs` command for gateway log access ([#1105](https://github.com/nearai/ironclaw/pull/1105))
- add Feishu/Lark WASM channel plugin ([#1110](https://github.com/nearai/ironclaw/pull/1110))
- add Criterion benchmarks for safety layer hot paths ([#836](https://github.com/nearai/ironclaw/pull/836))
- *(routines)* human-readable cron schedule summaries in web UI ([#1154](https://github.com/nearai/ironclaw/pull/1154))
- *(web)* add follow-up suggestion chips and ghost text ([#1156](https://github.com/nearai/ironclaw/pull/1156))
- *(ci)* include commit history in staging promotion PRs ([#952](https://github.com/nearai/ironclaw/pull/952))
- *(tools)* add reusable sensitive JSON redaction helper ([#457](https://github.com/nearai/ironclaw/pull/457))
- configurable hybrid search fusion strategy ([#234](https://github.com/nearai/ironclaw/pull/234))
- *(cli)* add cron subcommand for managing scheduled routines ([#1017](https://github.com/nearai/ironclaw/pull/1017))
- adds context-llm tool support ([#616](https://github.com/nearai/ironclaw/pull/616))
- *(web-chat)* add hover copy button for user/assistant messages ([#948](https://github.com/nearai/ironclaw/pull/948))
- add Slack approval buttons for tool execution in DMs ([#796](https://github.com/nearai/ironclaw/pull/796))
- enhance HTTP tool parameter parsing ([#911](https://github.com/nearai/ironclaw/pull/911))
- *(routines)* enable tool access in lightweight routine execution ([#257](https://github.com/nearai/ironclaw/pull/257)) ([#730](https://github.com/nearai/ironclaw/pull/730))
- add MiniMax as a built-in LLM provider ([#940](https://github.com/nearai/ironclaw/pull/940))
- *(cli)* add `ironclaw channels list` subcommand ([#933](https://github.com/nearai/ironclaw/pull/933))
- *(cli)* add `ironclaw skills list/search/info` subcommands ([#918](https://github.com/nearai/ironclaw/pull/918))
- add cargo-deny for supply chain safety ([#834](https://github.com/nearai/ironclaw/pull/834))
- *(setup)* display ASCII art banner during onboarding ([#851](https://github.com/nearai/ironclaw/pull/851))
- *(extensions)* unify auth and configure into single entrypoint ([#677](https://github.com/nearai/ironclaw/pull/677))
- *(i18n)* Add internationalization support with Chinese and English translations ([#929](https://github.com/nearai/ironclaw/pull/929))
- Import OpenClaw memory, history and settings ([#903](https://github.com/nearai/ironclaw/pull/903))

### Fixed

- jobs limit ([#1274](https://github.com/nearai/ironclaw/pull/1274))
- misleading UI message ([#1265](https://github.com/nearai/ironclaw/pull/1265))
- bump channel registry versions for promotion ([#1264](https://github.com/nearai/ironclaw/pull/1264))
- cover staging CI all-features and routine batch regressions ([#1256](https://github.com/nearai/ironclaw/pull/1256))
- resolve merge conflict fallout and missing config fields
- web/CLI routine mutations do not refresh live event trigger cache ([#1255](https://github.com/nearai/ironclaw/pull/1255))
- *(jobs)* make completed->completed transition idempotent to prevent race errors ([#1068](https://github.com/nearai/ironclaw/pull/1068))
- *(llm)* persist refreshed Anthropic OAuth token after Keychain re-read ([#1213](https://github.com/nearai/ironclaw/pull/1213))
- *(worker)* prevent orphaned tool_results and fix parallel merging ([#1069](https://github.com/nearai/ironclaw/pull/1069))
- Telegram bot token validation fails intermittently (HTTP 404) ([#1166](https://github.com/nearai/ironclaw/pull/1166))
- *(security)* prevent metadata spoofing of internal job monitor flag ([#1195](https://github.com/nearai/ironclaw/pull/1195))
- *(security)* default webhook server to loopback when tunnel is configured ([#1194](https://github.com/nearai/ironclaw/pull/1194))
- *(auth)* avoid false success and block chat during pending auth ([#1111](https://github.com/nearai/ironclaw/pull/1111))
- *(config)* unify ChannelsConfig resolution to env > settings > default ([#1124](https://github.com/nearai/ironclaw/pull/1124))
- *(web-chat)* normalize chat copy to plain text ([#1114](https://github.com/nearai/ironclaw/pull/1114))
- *(skill)* treat empty url param as absent when installing skills ([#1128](https://github.com/nearai/ironclaw/pull/1128))
- preserve AuthError type in oauth_http_client cache ([#1152](https://github.com/nearai/ironclaw/pull/1152))
- *(web)* prevent Safari IME composition Enter from sending message ([#1140](https://github.com/nearai/ironclaw/pull/1140))
- *(mcp)* handle 400 auth errors, clear auth mode after OAuth, trim tokens ([#1158](https://github.com/nearai/ironclaw/pull/1158))
- eliminate panic paths in production code ([#1184](https://github.com/nearai/ironclaw/pull/1184))
- N+1 query pattern in event trigger loop (routine_engine) ([#1163](https://github.com/nearai/ironclaw/pull/1163))
- *(llm)* add stop_sequences parity for tool completions ([#1170](https://github.com/nearai/ironclaw/pull/1170))
- *(channels)* use live owner binding during wasm hot activation ([#1171](https://github.com/nearai/ironclaw/pull/1171))
- Non-transactional multi-step context updates between metadata/to… ([#1161](https://github.com/nearai/ironclaw/pull/1161))
- *(webhook)* avoid lock-held awaits in server lifecycle paths ([#1168](https://github.com/nearai/ironclaw/pull/1168))
- Google Sheets returns 403 PERMISSION_DENIED after completing OAuth ([#1164](https://github.com/nearai/ironclaw/pull/1164))
- HTTP webhook secret transmitted in request body rather than via header, docs inconsistency and security concern ([#1162](https://github.com/nearai/ironclaw/pull/1162))
- *(ci)* exclude ironclaw_safety from release automation ([#1146](https://github.com/nearai/ironclaw/pull/1146))
- *(registry)* bump versions for github, web-search, and discord extensions ([#1106](https://github.com/nearai/ironclaw/pull/1106))
- *(mcp)* address 14 audit findings across MCP module ([#1094](https://github.com/nearai/ironclaw/pull/1094))
- *(http)* replace .expect() with match in webhook handler ([#1133](https://github.com/nearai/ironclaw/pull/1133))
- *(time)* treat empty timezone string as absent ([#1127](https://github.com/nearai/ironclaw/pull/1127))
- 5 critical/high-priority bugs (auth bypass, relay failures, unbounded recursion, context growth) ([#1083](https://github.com/nearai/ironclaw/pull/1083))
- *(ci)* checkout promotion PR head for metadata refresh ([#1097](https://github.com/nearai/ironclaw/pull/1097))
- *(ci)* add missing attachments field and crates/ dir to Dockerfiles ([#1100](https://github.com/nearai/ironclaw/pull/1100))
- *(registry)* bump telegram channel version for capabilities change ([#1064](https://github.com/nearai/ironclaw/pull/1064))
- *(ci)* repair staging promotion workflow behavior ([#1091](https://github.com/nearai/ironclaw/pull/1091))
- *(wasm)* address #1086 review followups -- description hint and coercion safety ([#1092](https://github.com/nearai/ironclaw/pull/1092))
- *(ci)* repair staging-ci workflow parsing ([#1090](https://github.com/nearai/ironclaw/pull/1090))
- *(extensions)* fix lifecycle bugs + comprehensive E2E tests ([#1070](https://github.com/nearai/ironclaw/pull/1070))
- add tool_info schema discovery for WASM tools ([#1086](https://github.com/nearai/ironclaw/pull/1086))
- resolve bug_bash UX/logging issues (#1054 #1055 #1058) ([#1072](https://github.com/nearai/ironclaw/pull/1072))
- *(http)* fail closed when webhook secret is missing at runtime ([#1075](https://github.com/nearai/ironclaw/pull/1075))
- *(service)* set CLI_ENABLED=false in macOS launchd plist ([#1079](https://github.com/nearai/ironclaw/pull/1079))
- relax approval requirements for low-risk tools ([#922](https://github.com/nearai/ironclaw/pull/922))
- *(web)* make approval requests appear without page reload ([#996](https://github.com/nearai/ironclaw/pull/996)) ([#1073](https://github.com/nearai/ironclaw/pull/1073))
- *(routines)* run cron checks immediately on ticker startup ([#1066](https://github.com/nearai/ironclaw/pull/1066))
- *(web)* recompute cron next_fire_at when re-enabling routines ([#1080](https://github.com/nearai/ironclaw/pull/1080))
- *(memory)* reject absolute filesystem paths with corrective routing ([#934](https://github.com/nearai/ironclaw/pull/934))
- remove all inline event handlers for CSP script-src compliance ([#1063](https://github.com/nearai/ironclaw/pull/1063))
- *(mcp)* include OAuth state parameter in authorization URLs ([#1049](https://github.com/nearai/ironclaw/pull/1049))
- *(mcp)* open MCP OAuth in same browser as gateway ([#951](https://github.com/nearai/ironclaw/pull/951))
- *(deploy)* harden production container and bootstrap security ([#1014](https://github.com/nearai/ironclaw/pull/1014))
- release lock guards before awaiting channel send ([#869](https://github.com/nearai/ironclaw/pull/869)) ([#1003](https://github.com/nearai/ironclaw/pull/1003))
- *(registry)* use versioned artifact URLs and checksums for all WASM manifests ([#1007](https://github.com/nearai/ironclaw/pull/1007))
- *(setup)* preserve model selection on provider re-run ([#679](https://github.com/nearai/ironclaw/pull/679)) ([#987](https://github.com/nearai/ironclaw/pull/987))
- *(mcp)* attach session manager for non-OAuth HTTP clients ([#793](https://github.com/nearai/ironclaw/pull/793)) ([#986](https://github.com/nearai/ironclaw/pull/986))
- *(security)* migrate webhook auth to HMAC-SHA256 signature header ([#970](https://github.com/nearai/ironclaw/pull/970))
- *(security)* make unsafe env::set_var calls safe with explicit invariants ([#968](https://github.com/nearai/ironclaw/pull/968))
- *(security)* require explicit SANDBOX_ALLOW_FULL_ACCESS to enable FullAccess policy ([#967](https://github.com/nearai/ironclaw/pull/967))
- *(security)* add Content-Security-Policy header to web gateway ([#966](https://github.com/nearai/ironclaw/pull/966))
- *(test)* stabilize openai compat oversized-body regression ([#839](https://github.com/nearai/ironclaw/pull/839))
- *(ci)* disambiguate WASM bundle filenames to prevent tool/channel collision ([#964](https://github.com/nearai/ironclaw/pull/964))
- *(setup)* validate channel credentials during setup ([#684](https://github.com/nearai/ironclaw/pull/684))
- drain tunnel pipes to prevent zombie process ([#735](https://github.com/nearai/ironclaw/pull/735))
- *(mcp)* header safety validation and Authorization conflict bug from #704 ([#752](https://github.com/nearai/ironclaw/pull/752))
- *(agent)* block thread_id-based context pollution across users ([#760](https://github.com/nearai/ironclaw/pull/760))
- *(mcp)* stdio/unix transports skip initialize handshake ([#890](https://github.com/nearai/ironclaw/pull/890)) ([#935](https://github.com/nearai/ironclaw/pull/935))
- *(setup)* drain residual events and filter key kind in onboard prompts ([#937](https://github.com/nearai/ironclaw/pull/937)) ([#949](https://github.com/nearai/ironclaw/pull/949))
- *(security)* load WASM tool description and schema from capabilities.json ([#520](https://github.com/nearai/ironclaw/pull/520))
- *(security)* resolve DNS once and reuse for SSRF validation to prevent rebinding ([#518](https://github.com/nearai/ironclaw/pull/518))
- *(security)* replace regex HTML sanitizer with DOMPurify to prevent XSS ([#510](https://github.com/nearai/ironclaw/pull/510))
- *(ci)* improve Claude Code review reliability ([#955](https://github.com/nearai/ironclaw/pull/955))
- *(ci)* run gated test jobs during staging CI ([#956](https://github.com/nearai/ironclaw/pull/956))
- *(ci)* prevent staging-ci tag failure and chained PR auto-close ([#900](https://github.com/nearai/ironclaw/pull/900))
- *(ci)* WASM WIT compat sqlite3 duplicate symbol conflict ([#953](https://github.com/nearai/ironclaw/pull/953))
- resolve deferred review items from PRs #883, #848, #788 ([#915](https://github.com/nearai/ironclaw/pull/915))
- *(web)* improve UX readability and accessibility in chat UI ([#910](https://github.com/nearai/ironclaw/pull/910))

### Other

- Fix Telegram auto-verify flow and routing ([#1273](https://github.com/nearai/ironclaw/pull/1273))
- *(e2e)* fix approval waiting regression coverage ([#1270](https://github.com/nearai/ironclaw/pull/1270))
- isolate heavy integration tests ([#1266](https://github.com/nearai/ironclaw/pull/1266))
- Merge branch 'main' into fix/resolve-conflicts
- Refactor owner scope across channels and fix default routing fallback ([#1151](https://github.com/nearai/ironclaw/pull/1151))
- *(extensions)* document relay manager init order ([#928](https://github.com/nearai/ironclaw/pull/928))
- *(setup)* extract init logic from wizard into owning modules ([#1210](https://github.com/nearai/ironclaw/pull/1210))
- mention MiniMax as built-in provider in all READMEs ([#1209](https://github.com/nearai/ironclaw/pull/1209))
- Fix schema-guided tool parameter coercion ([#1143](https://github.com/nearai/ironclaw/pull/1143))
- Make no-panics CI check test-aware ([#1160](https://github.com/nearai/ironclaw/pull/1160))
- *(mcp)* avoid reallocating SSE buffer on each chunk ([#1153](https://github.com/nearai/ironclaw/pull/1153))
- *(routines)* avoid full message history clone each tool iteration ([#1172](https://github.com/nearai/ironclaw/pull/1172))
- *(registry)* align manifest versions with published artifacts ([#1169](https://github.com/nearai/ironclaw/pull/1169))
- remove __pycache__ from repo and add to .gitignore ([#1177](https://github.com/nearai/ironclaw/pull/1177))
- *(registry)* move MCP servers from code to JSON manifests ([#1144](https://github.com/nearai/ironclaw/pull/1144))
- improve routine schema guidance ([#1089](https://github.com/nearai/ironclaw/pull/1089))
- add event-trigger routine e2e coverage ([#1088](https://github.com/nearai/ironclaw/pull/1088))
- enforce no .unwrap(), .expect(), or assert!() in production code ([#1087](https://github.com/nearai/ironclaw/pull/1087))
- periodic sync main into staging (resolved conflicts) ([#1098](https://github.com/nearai/ironclaw/pull/1098))
- fix formatting in cli/mod.rs and mcp/auth.rs ([#1071](https://github.com/nearai/ironclaw/pull/1071))
- Expose the shared agent session manager via AppComponents ([#532](https://github.com/nearai/ironclaw/pull/532))
- *(agent)* remove unnecessary Worker re-export ([#923](https://github.com/nearai/ironclaw/pull/923))
- Fix UTF-8 unsafe truncation in WASM emit_message ([#1015](https://github.com/nearai/ironclaw/pull/1015))
- extract safety module into ironclaw_safety crate ([#1024](https://github.com/nearai/ironclaw/pull/1024))
- Add Z.AI provider support for GLM-5 ([#938](https://github.com/nearai/ironclaw/pull/938))
- *(html_to_markdown)* refresh golden files after renderer bump ([#1016](https://github.com/nearai/ironclaw/pull/1016))
- Migrate GitHub webhook normalization into github tool ([#758](https://github.com/nearai/ironclaw/pull/758))
- Fix systemctl unit ([#472](https://github.com/nearai/ironclaw/pull/472))
- add Russian localization (README.ru.md) ([#850](https://github.com/nearai/ironclaw/pull/850))
- Add generic host-verified /webhook/tools/{tool} ingress ([#757](https://github.com/nearai/ironclaw/pull/757))

## [0.18.0](https://github.com/nearai/ironclaw/compare/v0.17.0...v0.18.0) - 2026-03-11

### Other

- Merge pull request #907 from nearai/staging-promote/b0214fef-22930316561
- promote staging to main (2026-03-10 15:19 UTC) ([#865](https://github.com/nearai/ironclaw/pull/865))
- Merge pull request #830 from nearai/staging-promote/3a2989d0-22888378864
- update WASM artifact SHA256 checksums [skip ci] ([#876](https://github.com/nearai/ironclaw/pull/876))

## [0.17.0](https://github.com/nearai/ironclaw/compare/v0.16.1...v0.17.0) - 2026-03-10

### Added

- *(llm)* per-provider unsupported parameter filtering (#749, #728) ([#809](https://github.com/nearai/ironclaw/pull/809))
- persist user_id in save_job and expose job_id on routine runs ([#709](https://github.com/nearai/ironclaw/pull/709))
- *(ci)* chained promotion PRs with multi-agent Claude review ([#776](https://github.com/nearai/ironclaw/pull/776))
- add background sandbox reaper for orphaned Docker containers ([#634](https://github.com/nearai/ironclaw/pull/634))
- *(wasm)* lazy schema injection on WASM tool errors ([#638](https://github.com/nearai/ironclaw/pull/638))
- add AWS Bedrock LLM provider via native Converse API ([#713](https://github.com/nearai/ironclaw/pull/713))
- full image support across all channels ([#725](https://github.com/nearai/ironclaw/pull/725))
- *(skills)* exclude_keywords veto in skill activation scoring ([#688](https://github.com/nearai/ironclaw/pull/688))
- *(mcp)* transport abstraction, stdio/UDS transports, and OAuth fixes ([#721](https://github.com/nearai/ironclaw/pull/721))
- add PID-based gateway lock to prevent multiple instances ([#717](https://github.com/nearai/ironclaw/pull/717))
- configurable LLM request timeout via LLM_REQUEST_TIMEOUT_SECS ([#615](https://github.com/nearai/ironclaw/pull/615)) ([#630](https://github.com/nearai/ironclaw/pull/630))
- *(timezone)* add timezone-aware session context ([#671](https://github.com/nearai/ironclaw/pull/671))
- *(setup)* Anthropic OAuth onboarding with setup-token support ([#384](https://github.com/nearai/ironclaw/pull/384))
- *(llm)* add Google Gemini, AWS Bedrock, io.net, Mistral, Yandex, and Cloudflare WS AI providers ([#676](https://github.com/nearai/ironclaw/pull/676))
- unified thread model for web gateway ([#607](https://github.com/nearai/ironclaw/pull/607))
- WASM channel attachments with LLM pipeline integration ([#596](https://github.com/nearai/ironclaw/pull/596))
- enable Anthropic prompt caching via automatic cache_control injection ([#660](https://github.com/nearai/ironclaw/pull/660))
- *(routines)* approval context for autonomous job execution ([#577](https://github.com/nearai/ironclaw/pull/577))
- *(llm)* declarative provider registry ([#618](https://github.com/nearai/ironclaw/pull/618))
- *(gateway)* show IronClaw version in status popover [skip-regression-check] ([#636](https://github.com/nearai/ironclaw/pull/636))
- Wire memory hygiene retention policy into heartbeat loop ([#629](https://github.com/nearai/ironclaw/pull/629))

### Fixed

- *(ci)* run fmt + clippy on staging PRs, skip Windows clippy [skip-regression-check] ([#802](https://github.com/nearai/ironclaw/pull/802))
- *(ci)* clean up staging pipeline — remove hacks, skip redundant checks [skip-regression-check] ([#794](https://github.com/nearai/ironclaw/pull/794))
- *(ci)* secrets can't be used in step if conditions [skip-regression-check] ([#787](https://github.com/nearai/ironclaw/pull/787))
- prevent irreversible context loss when compaction archive write fails ([#754](https://github.com/nearai/ironclaw/pull/754))
- button styles ([#637](https://github.com/nearai/ironclaw/pull/637))
- *(mcp)* JSON-RPC spec compliance — flexible id, correct notification format ([#685](https://github.com/nearai/ironclaw/pull/685))
- preserve tool-call history across thread hydration ([#568](https://github.com/nearai/ironclaw/pull/568)) ([#670](https://github.com/nearai/ironclaw/pull/670))
- CLI commands ignore runtime DATABASE_BACKEND when both features compiled ([#740](https://github.com/nearai/ironclaw/pull/740))
- *(web)* prevent fetch error when hostname is an IP address in TEE check ([#672](https://github.com/nearai/ironclaw/pull/672))
- add timezone conversion support to time tool ([#687](https://github.com/nearai/ironclaw/pull/687))
- standardize libSQL timestamps as RFC 3339 UTC ([#683](https://github.com/nearai/ironclaw/pull/683))
- *(docker)* bind postgres to localhost only ([#686](https://github.com/nearai/ironclaw/pull/686))
- *(repl)* skip /quit on EOF when stdin is not a TTY ([#724](https://github.com/nearai/ironclaw/pull/724))
- *(web)* prevent Enter key from sending message during IME composition ([#715](https://github.com/nearai/ironclaw/pull/715))
- *(config)* init_secrets no longer overwrites entire config ([#726](https://github.com/nearai/ironclaw/pull/726)) ([#734](https://github.com/nearai/ironclaw/pull/734))
- *(setup)* preserve model name when re-running onboarding with same provider ([#600](https://github.com/nearai/ironclaw/pull/600)) ([#694](https://github.com/nearai/ironclaw/pull/694))
- *(setup)* initialize secrets crypto for env-var security option ([#666](https://github.com/nearai/ironclaw/pull/666)) ([#706](https://github.com/nearai/ironclaw/pull/706))
- persist /model selection across restarts ([#707](https://github.com/nearai/ironclaw/pull/707))
- *(routines)* resolve message tool channel/target from per-job metadata ([#708](https://github.com/nearai/ironclaw/pull/708))
- sanitize HTML error bodies from MCP servers to prevent web UI white screen ([#263](https://github.com/nearai/ironclaw/pull/263)) ([#656](https://github.com/nearai/ironclaw/pull/656))
- prevent Instant duration overflow on Windows ([#657](https://github.com/nearai/ironclaw/pull/657)) ([#664](https://github.com/nearai/ironclaw/pull/664))
- enable libsql remote + tls features for Turso cloud sync ([#587](https://github.com/nearai/ironclaw/pull/587))
- *(tests)* replace hardcoded /tmp paths with tempdir + add 300 unit tests ([#659](https://github.com/nearai/ironclaw/pull/659))
- *(llm)* nudge LLM when it expresses tool intent without calling tools ([#653](https://github.com/nearai/ironclaw/pull/653))
- *(llm)* report zero cost for OpenRouter free-tier models ([#463](https://github.com/nearai/ironclaw/pull/463)) ([#613](https://github.com/nearai/ironclaw/pull/613))
- reliable network tests and improved tool error messages ([#626](https://github.com/nearai/ironclaw/pull/626))
- *(wasm)* use per-engine cache dirs on Windows to avoid file lock error ([#624](https://github.com/nearai/ironclaw/pull/624))
- *(libsql)* support flexible embedding dimensions ([#534](https://github.com/nearai/ironclaw/pull/534))

### Other

- Restructure CLAUDE.md into modular rules + add pr-shepherd command ([#750](https://github.com/nearai/ironclaw/pull/750))
- make src/llm/ self-contained for crate extraction ([#767](https://github.com/nearai/ironclaw/pull/767))
- add simplified Chinese (zh-CN) README translation ([#488](https://github.com/nearai/ironclaw/pull/488))
- *(job)* cover job tool validation and state transitions ([#681](https://github.com/nearai/ironclaw/pull/681))
- *(agent)* wire TestRig job tools through the scheduler ([#716](https://github.com/nearai/ironclaw/pull/716))
- Fix single-message mode to exit after one turn when background channels are enabled ([#719](https://github.com/nearai/ironclaw/pull/719))
- remove dead code ([#648](https://github.com/nearai/ironclaw/pull/648)) ([#703](https://github.com/nearai/ironclaw/pull/703))
- add reviewer-feedback guardrails (CLAUDE.md, pre-commit hook, skill) ([#665](https://github.com/nearai/ironclaw/pull/665))
- update WASM artifact SHA256 checksums [skip ci] ([#631](https://github.com/nearai/ironclaw/pull/631))
- add explanatory comments to coverage workflow ([#610](https://github.com/nearai/ironclaw/pull/610))
- build system prompt once per turn, skip tools on force-text ([#583](https://github.com/nearai/ironclaw/pull/583))
- add comprehensive subdirectory CLAUDE.md files and update root ([#589](https://github.com/nearai/ironclaw/pull/589))
- Improve test infrastructure: StubChannel, gateway helpers, security tests, search edge cases ([#623](https://github.com/nearai/ironclaw/pull/623))
- *(workspace)* regression test for document_path in search results ([#509](https://github.com/nearai/ironclaw/pull/509))

### Added

- AWS Bedrock LLM provider via native Converse API with IAM and SSO auth support (feature-gated: `--features bedrock`)

## [0.16.1](https://github.com/nearai/ironclaw/compare/v0.16.0...v0.16.1) - 2026-03-06

### Fixed

- revert WASM artifact SHA256 checksums to null ([#627](https://github.com/nearai/ironclaw/pull/627))

## [0.16.0](https://github.com/nearai/ironclaw/compare/v0.15.0...v0.16.0) - 2026-03-06

### Added

- *(e2e)* extensions tab tests, CI parallelization, and 3 production bug fixes ([#584](https://github.com/nearai/ironclaw/pull/584))
- WASM extension versioning with WIT compat checks ([#592](https://github.com/nearai/ironclaw/pull/592))
- Add HMAC-SHA256 webhook signature validation for Slack ([#588](https://github.com/nearai/ironclaw/pull/588))
- restart ([#531](https://github.com/nearai/ironclaw/pull/531))
- merge http/web_fetch tools, add tool output stash for large responses ([#578](https://github.com/nearai/ironclaw/pull/578))
- integrate 13-dimension complexity scorer into smart routing ([#529](https://github.com/nearai/ironclaw/pull/529))

### Fixed

- *(llm)* fix reasoning model response parsing bugs ([#564](https://github.com/nearai/ironclaw/pull/564)) ([#580](https://github.com/nearai/ironclaw/pull/580))
- *(ci)* fix three coverage workflow failures ([#597](https://github.com/nearai/ironclaw/pull/597))
- Telegram channel accepts group messages from all users if owner_… ([#590](https://github.com/nearai/ironclaw/pull/590))
- *(ci)* anchor coverage/ gitignore rule to repo root ([#591](https://github.com/nearai/ironclaw/pull/591))
- *(security)* use OsRng for all security-critical key and token generation ([#519](https://github.com/nearai/ironclaw/pull/519))
- prevent concurrent memory hygiene passes and Windows file lock errors ([#535](https://github.com/nearai/ironclaw/pull/535))
- sort tool_definitions() for deterministic LLM tool ordering ([#582](https://github.com/nearai/ironclaw/pull/582))
- *(ci)* persist all cargo-llvm-cov env vars for E2E coverage ([#559](https://github.com/nearai/ironclaw/pull/559))

### Other

- *(llm)* complete response cache — set_model invalidation, stats logging, sync mutex ([#290](https://github.com/nearai/ironclaw/pull/290))
- add 29 E2E trace tests for issues #571-575 ([#593](https://github.com/nearai/ironclaw/pull/593))
- add 26 tests for multi-thread safety, db CRUD, concurrency, errors ([#442](https://github.com/nearai/ironclaw/pull/442))
- update WASM artifact SHA256 checksums [skip ci] ([#560](https://github.com/nearai/ironclaw/pull/560))
- add WIT compatibility tests for WASM extensions ([#586](https://github.com/nearai/ironclaw/pull/586))
- Trajectory benchmarks and e2e trace test rig ([#553](https://github.com/nearai/ironclaw/pull/553))

## [0.15.0](https://github.com/nearai/ironclaw/compare/v0.14.0...v0.15.0) - 2026-03-04

### Added

- *(oauth)* route callbacks through web gateway for hosted instances ([#555](https://github.com/nearai/ironclaw/pull/555))
- *(web)* show error details for failed tool calls ([#490](https://github.com/nearai/ironclaw/pull/490))
- *(extensions)* improve auth UX and add load-time validation ([#536](https://github.com/nearai/ironclaw/pull/536))
- add local-test skill and Dockerfile.test for web gateway testing ([#524](https://github.com/nearai/ironclaw/pull/524))

### Fixed

- *(security)* restrict query-token auth to SSE endpoints only ([#528](https://github.com/nearai/ironclaw/pull/528))
- *(ci)* flush profraw coverage data in E2E teardown ([#550](https://github.com/nearai/ironclaw/pull/550))
- *(wasm)* coerce string parameters to schema-declared types ([#498](https://github.com/nearai/ironclaw/pull/498))
- *(agent)* strip leaked [Called tool ...] text from responses ([#497](https://github.com/nearai/ironclaw/pull/497))
- *(web)* reset job list UI on restart failure ([#499](https://github.com/nearai/ironclaw/pull/499))
- *(security)* replace .unwrap() panics in pairing store with proper error handling ([#515](https://github.com/nearai/ironclaw/pull/515))

### Other

- Fix UTF-8 unsafe truncation in sandbox log capture ([#359](https://github.com/nearai/ironclaw/pull/359))
- enhance coverage with feature matrix, postgres, and E2E ([#523](https://github.com/nearai/ironclaw/pull/523))

## [0.14.0](https://github.com/nearai/ironclaw/compare/v0.13.1...v0.14.0) - 2026-03-04

### Added

- remove the okta tool ([#506](https://github.com/nearai/ironclaw/pull/506))
- add OAuth support for WASM tools in web gateway ([#489](https://github.com/nearai/ironclaw/pull/489))
- *(web)* fix jobs UI parity for non-sandbox mode ([#491](https://github.com/nearai/ironclaw/pull/491))
- *(workspace)* add TOOLS.md, BOOTSTRAP.md, and disk-to-DB import ([#477](https://github.com/nearai/ironclaw/pull/477))

### Fixed

- *(web)* mobile browser bar obscures chat input ([#508](https://github.com/nearai/ironclaw/pull/508))
- *(web)* assign unique thread_id to manual routine triggers ([#500](https://github.com/nearai/ironclaw/pull/500))
- *(web)* refresh routine UI after Run Now trigger ([#501](https://github.com/nearai/ironclaw/pull/501))
- *(skills)* use slug for skill download URL from ClawHub ([#502](https://github.com/nearai/ironclaw/pull/502))
- *(workspace)* thread document path through search results ([#503](https://github.com/nearai/ironclaw/pull/503))
- *(workspace)* import custom templates before seeding defaults ([#505](https://github.com/nearai/ironclaw/pull/505))
- use std::sync::RwLock in MessageTool to avoid runtime panic ([#411](https://github.com/nearai/ironclaw/pull/411))
- wire secrets store into all WASM runtime activation paths ([#479](https://github.com/nearai/ironclaw/pull/479))

### Other

- enforce regression tests for fix commits ([#517](https://github.com/nearai/ironclaw/pull/517))
- add code coverage with cargo-llvm-cov and Codecov ([#511](https://github.com/nearai/ironclaw/pull/511))
- Remove restart infrastructure, generalize WASM channel setup ([#493](https://github.com/nearai/ironclaw/pull/493))
