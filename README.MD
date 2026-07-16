# SpacetimeDB-memBUS

> [!WARNING]
> **2026-07-16 — R6 is a Windows x64 test candidate. Clean-package verification has passed, but project-lead testing, the approval-only six-hour soak and final release acceptance remain pending. It is not production-ready.**

SpacetimeDB-memBUS is an experimental, MOD-like extension for SpacetimeDB that lets independent database processes on the same Windows machine invoke approved destination reducers through authenticated Windows named shared memory. The short project name is **memBUS**.

It is not a second database engine. Shared memory carries framed requests, responses and bounded metadata only. Tables, transactions, pointers, WASM linear memory and `RelationalDB` objects remain process-local. Destination work enters the normal SpacetimeDB `ModuleHost`, reducer executor, transaction, durability and subscription path.

> **Current candidate:** `2.6.1-R6` — Windows x64, based on SpacetimeDB v2.6.1. The binary package is intended for controlled evaluation. It has passed automated clean-extraction correctness, but not the project lead's personal release test.

> **Production licensing notice:** SpacetimeDB 2.6.1 is distributed under the Business Source License 1.1. Its Additional Use Grant permits an application or service to use no more than **one SpacetimeDB instance in production** and excludes use as a Database Service. SpacetimeDB-memBUS is designed for multiple independent instances, so a production multi-instance topology requires a commercial licence or another applicable arrangement from Clockwork Laboratories. Review [`LICENSE.txt`](LICENSE.txt), the [upstream licence](https://github.com/clockworklabs/SpacetimeDB/blob/v2.6.1/LICENSE.txt), and obtain qualified advice for your deployment.

## Accepted Build 12 performance checkpoint

![SpacetimeDB-memBUS R6 history and current comparison](documentation/assets/spacetimedb-membus-r6-history-comparison.png)

Three fresh matched process pairs used 100 warm-up and 1,000 measured calls per path/pair with a two-byte payload and normal Windows scheduling:

| Boundary | memBUS P50/P95/P99 | Same-session persistent local HTTP P50/P95/P99 | Result |
|---|---:|---:|---:|
| prepared pre-send -> destination dispatch | `0.0249 / 0.0355 / 0.0455 ms` | `0.1674 / 0.2446 / 0.2942 ms` | memBUS `6.72x / 6.89x / 6.47x` faster |
| prepared pre-send -> committed ACK/response | `0.1197 / 0.1826 / 0.2245 ms` | `0.4084 / 0.6036 / 0.7779 ms` | memBUS `3.41x / 3.31x / 3.47x` faster |

All 3,000 operations per path committed. The comparator is persistent loopback HTTP, not HTTPS/TLS, and it is not a memBUS fallback. The explicit `<= 0.20 ms P99` target remains open at `0.2245 ms`. These are Build 12 measurements, not universal guarantees and not an automatic performance claim for any differently hashed rebuild. Open the [R6 chart](documentation/db-membus-benchmark-chart.html), [R1-R6 history](documentation/db-membus-release-history-chart.html), or [methodology](documentation/benchmarks.md).

## Why it exists

SpacetimeDB-memBUS was created for **my MMORPG project**, which uses colocated databases with explicit ownership. `PersistenceDB` owns durable character, inventory, equipment and progression truth. Regional GameWorld databases receive only thin, validated gameplay projections and return compact progression/loot operations. An ApiCoordinator owns cross-database workflows and idempotent recovery because no transaction spans two databases.

memBUS does not replace that coordinator. **The coordinator decides the workflow; memBUS carries approved commands, projections and commit-aware acknowledgements between colocated database processes with less transport overhead.**

![My MMORPG project's target multi-database topology](MMORPG_MEMBUS_TOPOLOGY.svg)

## One memBUS operation

```text
approved source procedure
-> configured local memBUS route
-> destination reducer through the normal SpacetimeDB execution path
-> destination transaction result
-> correlated response or explicit uncertainty
```

No arbitrary reducer gateway exists. Participants, route, schema and destination operation must all be approved. Detailed wire layout, host hooks and private implementation evidence are intentionally maintained in the private development repository.

## Core properties

- independent processes, databases and transactions remain isolated;
- only explicitly configured participants and destination operations are accepted;
- local transport, admission, payload and retry state are bounded;
- destination work follows the normal SpacetimeDB reducer and commit path;
- delivery is at least once with transaction-aware results and reconciliation;
- invalid, unauthorized or incompatible input fails closed;
- no automatic HTTP, TCP, named-pipe or ApiCoordinator fallback.

## Release package quick start

The local package assembly and `dist` directory are deliberately ignored by the public repository. Download the separately published test-candidate asset, verify its adjacent SHA-256 file, extract it, then run from the extracted `2.6.1-R6` folder:

```powershell
Set-ExecutionPolicy -Scope Process Bypass
.\Test-PackageIntegrity.ps1
.\Start-Demo.ps1
```

After both black endpoint consoles show magenta `MEMBUS ... runtime ready` messages:

```powershell
.\Run-MemBus-Test.ps1
```

Expected result:

```text
"TransactionCommitted"
```

Stop and reset generated state:

```powershell
.\Stop-Demo.ps1
.\Reset-Demo.ps1
```

See [Getting started](documentation/getting-started.md).

## Commands at a glance

| Surface | Command/function | Purpose |
|---|---|---|
| Package | `Start-Demo.ps1` | Verify, provision, restore and open both endpoint consoles |
| Package | `Run-MemBus-Test.ps1` | Execute one committed R6 alpha -> beta call |
| Package | `Stop-Demo.ps1` | Gracefully stop the packaged endpoints |
| Package | `Reset-Demo.ps1` | Remove generated package-local state after shutdown |
| C# | `ctx.MemBus.Call(...)` | Invoke one topology-approved destination operation from a procedure |

### Example call

```csharp
var result = ctx.MemBus.Call(
    target: configuredTarget,
    channel: configuredChannel,
    reducer: approvedOperation,
    payload: serializedPayload,
    timeout: deadline,
    operationId: stableOperationId
);
```

The call is procedure-only. Target, channel and destination operation must already be permitted by topology; the API is not an arbitrary reducer gateway.

## Delivery contract

Initial delivery is **at least once**, not exactly once. Stable operation identity, destination idempotency and reconciliation prevent an uncertain result from being silently converted into success.

`TransactionCommitted` means the normal destination reducer transaction returned committed and subscriptions were published. It does not claim fsync. `TimedOut`/`Unknown` remain uncertain and require reconciliation; they are never converted into success.

## Revision path

- **R1:** initial Windows shared-memory proof of concept.
- **R2:** matched measurement, bounded admission and commit-aware result handling.
- **R3:** authenticated local routes and principal validation.
- **R4:** bounded uncertainty, reconciliation and restart recovery.
- **R5:** multi-route operational foundations and lifecycle hardening.
- **R6:** compact bounded operations, reduced allocation and lower-observer-effect profiling.

## Remaining acceptance work

- project-lead extracted-package test of the freshly rebuilt R6 candidate;
- six-hour release-candidate soak, started only with explicit project-lead approval;
- Windows service mode is not included in this interactive demo package; retained private evidence covers its engineering acceptance separately;
- repeated P99/P99.9, crash, soak, pressure and service-mode campaigns.

The project remains a focused same-machine transport. It will not add automatic HTTP fallback, arbitrary reducer execution, cross-process transactions, shared database objects or unbounded retries.

## Moving ahead

- **SpacetimeDB Core Reducer Optimization Research** — Measure reducer preparation, execution, commit and subscription costs before proposing any upstream-core change. Core modifications remain a separate, explicitly approved track.
- **Per-process CPU/Core Management** — Replace the experimental single `CpuIndex` setting with validated process and thread assignments for database, memBUS, Tokio, Rayon and background work.
- **CPU Topology Manager** — Detect logical and physical cores, SMT siblings, P/E classes, NUMA nodes and processor groups. Reject invalid or overlapping assignments and expose the accepted plan in diagnostics.
- **CPU Reservation Profiles** — Provide reviewed profiles for desktop development, server, NUMA, low-power and maximum-latency-performance deployments.
- **Runtime Affinity Inspector** — Report endpoint PID, process CPU sets, database/memBUS/Tokio placement, NUMA location and observed CPU migration.
- **Adaptive Receiver Profiles** — Add bounded power-saving, balanced, dedicated-core and auto wait strategies using measured HOT/WARM/COOLDOWN/PARKED transitions.
- **Queue Handoff Review** — Measure the existing blocking-waiter to bounded-Tokio handoff before considering a dedicated inbound executor or direct dispatch.
- **Drain Batching** — Drain bounded groups of ready frames per wake to improve burst throughput and tail latency without weakening per-route limits.
- **Commit-path ACK Audit** — Keep transaction-aware ACK creation directly on the committed reducer completion path and prove that no observer or extra task delays it.
- **Reducer Profiler and Advisor** — Attribute decode, lookup, mutation, commit and subscription cost per reducer, then suggest indexes, cloning reductions or fusion candidates without rewriting code automatically.
- **Cache-line and Wake Optimization** — Experiment with isolated producer/consumer cursors and signal only when a correctly synchronized consumer is parked. Any layout or wake-state change requires protocol compatibility and lost-wakeup proof.
- **Bounded Runtime Tuning** — Adapt hot-spin and drain bounds only inside administrator-defined limits using queue occupancy, arrival rate and tail-delay evidence.
- **Route Health and Timeline Diagnostics** — Expose route state, epochs, occupancy, in-flight work, failures and the full lifecycle of one operation ID.
- **Live-safe Configuration Reload** — Reload only operational limits, profiling and logging controls; identity, capability, schema, allowlist and protocol changes continue to require restart and handshake.
- **Packaged Benchmark and Failure Injection** — Ship repeatable percentile/throughput campaigns and controlled restart, corruption, timeout, duplicate and backpressure scenarios.
- **Topology Validator and Windows Service** — Validate collisions, reverse ACK routes, CPU allocations and reconciliation capacity, then complete SCM/Session 0 ACL and recovery acceptance.
- **Linux Backend and Coordinator SDK** — Preserve the core protocol while adding a Linux platform backend and typed local ApiCoordinator APIs with deadlines, cancellation and no fallback.

Planned release themes: **R7 — CPU and Receiver Control**, **R8 — Reducer and Commit Optimization**, **R9 — Production Operations**, and **R10 — Cross-platform and Ecosystem**. See the private development roadmap for implementation gates and exact acceptance criteria.

## Documentation

- [Documentation index](documentation/README.md)
- [Getting started](documentation/getting-started.md)
- [Architecture](documentation/architecture.md)
- [End-to-end path](documentation/end-to-end.md)
- [Configuration](documentation/configuration.md)
- [Command reference](documentation/commands.md)
- [Protocol and delivery](documentation/protocol.md)
- [Operations](documentation/operations.md)
- [Benchmarks](documentation/benchmarks.md)
- [Troubleshooting](documentation/troubleshooting.md)
- [Upgrade strategy](documentation/upgrade-guide.md)

## License and support

Read [LICENSE.txt](LICENSE.txt), [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md), [Security](SECURITY.md) and [Support](SUPPORT.md) before use or redistribution. Source code is not included at this project stage.
