# Changelog

> Version baseline: IronClaw v0.18.0 (`v0.18.0` tag snapshot)
> Scope: mirrored release changelog entries for the latest relevant released versions

All notable changes to this project are documented upstream in `nearai/ironclaw`. This mirror intentionally focuses on the latest released versions relevant to the current docs baseline.

## [Unreleased]

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
