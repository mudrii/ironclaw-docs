# Changelog

> Version baseline: IronClaw v0.16.1 (`v0.16.1` tag snapshot)
> Scope: mirrored release changelog entries for the latest relevant released versions

All notable changes to this project are documented upstream in `nearai/ironclaw`. This mirror intentionally focuses on the latest released versions relevant to the current docs baseline.

## [Unreleased]

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
