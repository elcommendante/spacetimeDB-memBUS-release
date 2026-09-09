# SpacetimeDB-memBUS public documentation

**Current candidate:** `2.10.0-R1`, Windows x64, SpacetimeDB v2.10.0 — one fork build with three components: **SpacetimeDB-memBUS** (shared-memory transport), **SpacetimeDB-memBUS-ephemeral** (ephemeral tables) and **SpacetimeDB-memBUS-relay** (relay channel). Automated clean-package verification has passed; project-lead testing, final acceptance and the six-hour soak remain pending.

- [Getting started](getting-started.md) — package verification, start, the three tests, stop and reset.
- [Architecture](architecture.md) — process boundaries and what each component owns.
- [Ephemeral tables](ephemeral-tables.md) — contract, declaration, migration, verification, measurements.
- [Relay channel](relay-channel.md) — contract, messages, configuration, client and bridge usage, measurements.
- [Operation lifecycle (memBUS)](end-to-end.md) — public request, destination transaction and result semantics.
- [Configuration](configuration.md) — topology v5, relay configuration, bounds and the authenticated route contract.
- [Commands](commands.md) — package, standalone, security utility, module and sample operations.
- [Delivery and security](protocol.md) — stable public behaviour of the three lanes.
- [Operations](operations.md) — lifecycle, recovery, reconciliation and failure handling.
- [Benchmarks](benchmarks.md) — 2.10.0-R1 ephemeral/relay campaign, R6 transport checkpoint, caveats.
- [MMORPG use case](mmorpg-use-case.md) — persistence/gameplay/coordinator boundary and what moved to Ephemeral + Relay.
- [Troubleshooting](troubleshooting.md) — typed failures and package diagnostics.
- [Development](development.md) — public quality and verification model.
- [Upgrade guide](upgrade-guide.md) — clean baseline and intentional port workflow (2.6.1 → 2.8.3 → 2.10.0).
- [Licensing](licensing.md) — upstream production restrictions.

The public repository contains documentation and separately uploaded binary assets, not development source or private evidence. The `2.6.1-R6` numbers are kept in [Benchmarks](benchmarks.md) and [CHANGELOG.md](../CHANGELOG.md).
