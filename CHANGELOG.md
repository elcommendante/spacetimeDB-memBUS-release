# Release changelog

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
