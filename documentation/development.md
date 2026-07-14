# Development and verification

> **Source availability:** development source is not included in the public repository at this stage. This page documents the engineering and verification model behind the binary candidate; the commands below are maintainers' reference commands and cannot be run from the release-only checkout.

## Source ownership

```text
crates/db-membus/                 stable transport/protocol/platform
crates/db-membus-spacetimedb/     version-sensitive SpacetimeDB adapter
crates/bindings-csharp/Runtime/    procedure API and private FFI
modules/db-membus-e2e-cs/         reference outbox/inbox module
```

Thin existing-core hooks register the ABI, start the runtime, expose metrics/logging, and add standalone arguments. Keep protocol/ring/routing logic out of entrypoints.

## Exact v2.6 path

The adapter resolves `get_database_by_identity → leader → module → call_reducer`. `ReducerOutcome::Committed`, `Failed`, and `BudgetExceeded` map to explicit MemBus outcomes. Scheduler/HTTP internals are precedents, not alternate transports.

## Build discipline

- Rust 1.93.0 from `rust-toolchain.toml`;
- `lld-link` from repository Windows configuration;
- locked dependencies;
- no lockfile/toolchain/dependency update to fix a build;
- separate build output outside source;
- `-t:Rebuild` when switching C# `DB_MEMBUS` because incremental outputs may mix variants.

## Gates

```powershell
cargo fmt --all -- --check
cargo check --locked -p db-membus -p db-membus-spacetimedb
cargo test --locked -p db-membus
cargo test --locked -p db-membus-spacetimedb --test service
cargo clippy --locked -p db-membus -p db-membus-spacetimedb --all-targets --no-deps -- -D warnings
cargo check --locked -p spacetimedb-standalone
cargo build --locked --release -p spacetimedb-standalone --features db-membus
dotnet build modules\db-membus-e2e-cs\db-membus-e2e-cs.csproj -t:Rebuild -c Release -p:DB_MEMBUS=1
```

Then prove two real Windows processes, named mapping/events, reducer mutation, committed ACK, duplicate suppression, and any changed failure lifecycle.

## Documentation requirement

Protocol, configuration, integration hooks, tests, benchmarks, changelog, and progress evidence must change with behavior. “Compiles” is not a completion proof.

Public release acceptance additionally requires package-integrity verification, clean extraction, two real processes, authenticated route readiness, a committed sample, graceful shutdown and a clean reset. R6 still awaits the project lead's extracted-package acceptance test.
