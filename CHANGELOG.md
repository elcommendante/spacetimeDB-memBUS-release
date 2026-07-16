# Release changelog

## 2.6.1-R6 documentation boundary correction — 2026-07-16

- Reduced the public documentation to stable product behavior, package operation, measured results, supported scope and known limitations.
- Removed private ABI maps, exact internal operation tables, cryptographic construction details and source-level implementation guidance from the public surface.
- Corrected the clean-package and retained Windows service evidence status without changing the tested package assembly or ZIP.
- Added a private engineering index and junior onboarding guide to the development repository; those implementation details are not publicly distributed.

## 2.6.1-R6 history-chart correction — 2026-07-16

- Updated the release-history chart from the superseded July 14 R6 campaign to the accepted Build 12 results.
- Current chart values are memBUS `0.0249/0.0355 ms` transport and `0.1197/0.1826 ms` full P50/P95 versus persistent local HTTP `0.1674/0.2446 ms` and `0.4084/0.6036 ms`.
- Recalculated every current bar height and comparison multiplier; historical R1-R5 values remain unchanged.

## 2.6.1-R6 package correction — 2026-07-16

- Replaced the oversized working-state archive with a 289.21 MiB test candidate built from the exact working Build 12 seeds.
- Removed only regenerable Wasmtime cache, logs, lock files and PID state; both independent endpoint binaries and required database state remain included.
- Rejected a more aggressive seed reconstruction after it failed clean-extraction E2E with a WASM trap.
- Reverified package integrity, two-process startup, `TransactionCommitted`, destination inspection, graceful Ctrl+C shutdown and reset.
- Final ZIP SHA-256: `64dada35ddceb425d8a5b24fa24cdfce592dd01fc124a446a966d76e62cee602`.

## 2.6.1-R6 Build 12 test candidate — 2026-07-16

- Updated the public and bundled documentation to the accepted Build 12 checkpoint: memBUS `0.0249/0.0355/0.0455 ms` transport and `0.1197/0.1826/0.2245 ms` full P50/P95/P99.
- Recorded the matched persistent-local-HTTP comparator at `0.1674/0.2446/0.2942 ms` transport and `0.4084/0.6036/0.7779 ms` full P50/P95/P99.
- Prepared a Windows x64 test-candidate assembly with the exact accepted Build 12 seed archives and a freshly rebuilt current-source standalone/security utility.
- Kept binary identity honest: the packaged rebuild has a different SHA-256 than the retained Build 12 benchmark manifest and does not inherit benchmark acceptance by name.
- Added the upload-ready ZIP and adjacent SHA-256 to ignored `dist` only after clean-extraction integrity and two-process committed-call verification.
- Final production/release acceptance and the explicitly gated six-hour soak remain pending.

## 2.6.1-R6 repository surface — 2026-07-14

- Updated the repository README, public documentation index, release procedures, security/support guidance and issue templates for the R6 candidate.
- Replaced the stale R1 MMORPG benchmark link and numbers with the matched R6 campaign and current repository-tracked charts.
- Made the bundled benchmark report self-contained; it no longer references private build/test-evidence paths or stale pre-packaging state.
- Retained the local ignored `2.6.1-R6` package assembly while intentionally withholding the R6 ZIP and adjacent checksum from `dist` until the project lead's extracted-package test.
- No source code, private evidence, secrets or active database state was added to the public repository surface.

## 2.6.1-R6 candidate — 2026-07-14

Candidate Windows x64 binary distribution based on SpacetimeDB v2.6.1 and the accepted Build 10/R6 two-process evidence.

- Added binary application operation schema v2 with fixed `U128` operation IDs, fixed `U256` SHA-256 digests, source-scoped binary outbox/inbox rows and explicit v2 reconciliation operations.
- Added compact route request v3, reusable producer scratch, single-owned receive payload and lower-observer-effect profiling without changing authenticated Frame v2 or Clockwork Labs core.
- Retained authenticated local routes, integrity and replay protection, explicit destination allowlists, at-least-once delivery, bounded uncertainty, reconciliation and post-commit ACK.
- Accepted a real two-process v2 correctness campaign covering direct commit, duplicate idempotency, digest conflict, exact 3448-byte application maximum, max+1 rejection, peer loss, fresh PID/session/epoch restart and reconciliation.
- Measured five fresh matched process pairs: R6 `0.0250 / 0.0382 ms` transport and `0.2762 / 0.4615 ms` full P50/P95 versus same-session persistent local HTTP `0.1304 / 0.1755 ms` and `0.4731 / 0.6884 ms`.
- Added a candidate package with separate alpha/beta binaries, accepted clean seed pair 4, matching CLI, `membus-security`, package-relative PowerShell start/stop/reset/test scripts, checksums, flow diagram, R6 chart and R1-R6 history chart.
- Excluded bearer tokens, JWT/capability secrets, active database state, source code, private development evidence and the rejected seed experiments 5/6.

**Acceptance:** build/focused-test/real two-process v2/interactive matched benchmark evidence is accepted. The assembled ZIP still requires the project lead's personal extracted-package test. Four-process schema-v2 and elevated SCM/Session 0 remain open, so this is not a production release.

## Roadmap summary — 2026-07-11

- Added a compact production-readiness roadmap to the public README.
- Recorded the remaining security, recovery, payload, fanout, service-mode and operational acceptance work without expanding memBUS into a general network protocol.
- Removed stale R1 performance/integration wording from the current R2 README status section.

## Repository performance preview — 2026-07-11

- Added a repository-tracked PNG of the accepted R2 latency comparison.
- Embedded the chart near the top of `README.MD` with the exact P50 multipliers, benchmark conditions and a non-guarantee qualification.
- Kept the R2 ZIP unchanged because this is a repository presentation asset, not a binary-package behavior change.

## 2.6.1-R2 — 2026-07-11

Optimized SpacetimeDB-memBUS Windows x64 proof-of-concept release based on SpacetimeDB v2.6.1.

- Replaced both standalone executables with accepted build `2.6.1 - 5` (`SHA-256 14F10E81E4B8D9FD2927C1C397CBCEE65FAC6EC057C1B694CE0CB4A0AC4591C7`).
- Added bounded nonblocking admission, one absolute request deadline, late-ACK capacity reconciliation, and a prewarmed generation-safe destination host cache.
- Removed redundant hot-path decoding/allocation while preserving explicit topology, reducer allowlists, authorization, idempotency, normal reducer execution, transaction commit and typed ACK semantics.
- Updated the bundled benchmark chart and public documentation with the final matched 100-warm-up/1,000-measured campaign.
- Published the same chart under `documentation/` so it renders from the repository even though local package assemblies and `dist/` assets remain ignored.
- Measured memBUS at `0.0206 / 0.0348 ms` P50/P95 transport and `0.2931 / 0.4301 ms` full commit+ACK.
- Measured persistent local HTTP at `0.2885 / 0.3738 ms` P50/P95 transport and `0.6558 / 0.8629 ms` full commit+response.
- Disabled process-wide one-core affinity by default because it materially worsened P95; explicit `-CpuIndex` experiments remain available.
- Retained the verified clean seeds, bundled matching CLI, PowerShell-only launch/test flow, manifests and release-local paths.
- Added a prominent production-licensing notice: the SpacetimeDB 2.6.1 Additional Use Grant covers no more than one production instance, while the intended memBUS topology uses multiple independent instances.

Known limitations remain: Windows x64/same-session only, at-least-once rather than exactly-once, committed ACK is not an fsync claim, Session 0 and real fanout are not accepted, and the demonstration C# fixture is accepted through 1,024-byte payloads but not 2,048/3,000 bytes.

## PowerShell-only launch and black console — 2026-07-11

- Removed both temporary CMD wrappers from the release and ZIP.
- Added root `Run-MemBus-Test.ps1`; `Start-Demo.ps1` remains the only coordinated startup entrypoint.
- Alpha and beta starters now force a black background, white default text, and endpoint-specific window titles; runtime `MEMBUS` labels remain magenta.
- Rebuilt and reverified the final ZIP through the PS1 entrypoints. SHA-256: `e6169353fcecb8f6ad4982e1a33d641a249848c1b4ec59c7f28455a7ac1002be`.

## End-to-end ZIP package — 2026-07-11

- Added `INSTRUCTION.md` with an exact extract, start, readiness, test, stop and reset workflow.
- Replaced the temporary CMD wrappers with PowerShell-only `Start-Demo.ps1` and `Run-MemBus-Test.ps1` entrypoints.
- Endpoint starter scripts now force a black console background and white default text while runtime `MEMBUS` labels remain magenta.
- Enabled visible `MEMBUS=debug` logging in both clean endpoint seeds.
- Added a reproducible Windows x64 ZIP distribution and adjacent SHA-256 checksum.
- Verified the ZIP after extraction with two real standalone processes and a committed cross-process memBUS operation.

## memBUS console logging fix — 2026-07-11

- Added the missing `membus=debug` filter directive to both clean database seeds.
- Future first-run restores now display `membus.*` transport events through the magenta `MEMBUS` formatter.
- Existing extracted data requires one restart after adding the directive because the running release does not reload this filter immediately.

## Clone-ready portable package — 2026-07-11

- Replaced 1.33 GiB of active runtime database state with two clean, SHA-256-pinned seed archives of approximately 43.5 MiB each.
- Added verified first-run seed restoration while keeping generated `data/`, keys, logs, caches, dumps and PID files outside version control.
- Added a bundled matching CLI and release-local JWT key generation; the MemBus sample no longer depends on a global installation or stale user token.
- Added `Start-Demo.ps1` to restore both endpoints and launch alpha/beta inside the strict route handshake window.
- Regenerated demonstration identities and retained AUTH=A+ by rebuilding the fixture with the exact new alpha identity.
- Verified a simulated fresh clone: seed restore, local keys, two listeners, real cross-process call, `TransactionCommitted`, destination inbox row and effect mutation.

## MMORPG architecture documentation — 2026-07-11

- Added a public, project-neutral explanation of why SpacetimeDB-memBUS exists for **my MMORPG project**.
- Added an English topology diagram connecting PersistenceDB, two GameWorld region databases, StatisticsDB, MarketplaceDB, ApiCoordinator, and memBUS.
- Documented the thin equipment/stat projection, progression and loot return flows, the coordinator boundary, and the distinction between current responsibilities and planned services.
- Linked the use case to the packaged benchmark chart without claiming that shared memory replaces reducer authorization, transactions, or durability.

## Documentation update — 2026-07-11

- Standardized the public project name as **SpacetimeDB-memBUS**.
- Standardized the short name as **memBUS**.
- Preserved literal technical identifiers such as `db-membus`, `DB_MEMBUS`, `db_membus_0.1`, `--membus-*`, and `Local\DBMemBus.*`.

## 2.6.1-1 — 2026-07-11

Initial packaged SpacetimeDB-memBUS release based on SpacetimeDB v2.6.1.

### Included

- memBUS-enabled Windows x64 standalone binary;
- separate alpha and beta endpoint folders;
- matching persistent demonstration data and database identities;
- request/response topology with transaction acknowledgement and inbox result;
- CPU-affinity starters for logical CPUs 0 and 1;
- MemBus, local HTTP, and TLS-only HTTPS sample scripts;
- complete flow SVG and annotated topology template;
- static and release SHA-256 manifests;
- GitHub-oriented operator and developer documentation.

### Verified

- packaged alpha on `127.0.0.1:3910`, affinity mask 1;
- packaged beta on `127.0.0.1:3920`, affinity mask 2;
- MemBus example returned `TransactionCommitted`;
- explicit local HTTP example completed;
- HTTPS example rejected HTTP downgrade;
- identical standalone binary hashes and topology hashes;
- all packaged PowerShell files parsed successfully.

### Known limitations

- Windows x64 and same-machine transport only;
- bundled standalone listener is HTTP, not native TLS;
- at-least-once delivery, not exactly once;
- committed ACK does not claim fsync;
- full-call P50 optimization is still planned;
- crash-boundary, Session 0, real fanout, and modified-runtime graceful shutdown campaigns remain open.
