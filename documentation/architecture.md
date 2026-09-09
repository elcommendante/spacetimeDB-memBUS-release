# Architecture overview

SpacetimeDB-memBUS is one fork build of SpacetimeDB with three independent components. Each lives in its own crate on top of an unmodified upstream baseline and hooks the host in a small, ledgered set of places.

```text
                         ┌─────────────────────────── one database process ───────────────────────────┐
clients (WebSocket) ───► │ client API ── subscriptions ── ModuleHost/reducers ── datastore ── commit log │
                         │      │                              ▲                     │                  │
                         │      │  SpacetimeDB-Relay           │ bridge reducer      │ SpacetimeDB-Ephemeral
                         │      └─ channel hub, AOI grid ──────┘ (per interval)      └─ rows flagged ephemeral:
                         │         rate limits, fan-out                                 transactional + subscribed,
                         │         (never a transaction)                                never appended
                         │                              SpacetimeDB-memBUS                              │
                         │                              ▲ authenticated shared-memory route             │
                         └──────────────────────────────┼───────────────────────────────────────────────┘
                                                        ▼
                         ┌─────────────────────── another database process ────────────────────────────┐
                         │ approved destination reducer ── normal transaction ── commit-aware ACK       │
                         └────────────────────────────────────────────────────────────────────────────┘
```

## memBUS

```text
source database process
-> approved local memBUS route
-> destination database process
-> normal destination reducer and transaction
-> committed response or explicit uncertainty
```

Each process keeps its own database, memory, transactions, WASM runtime, durability state and lifecycle. Shared memory transports bounded message data only; it never exposes another process's tables, pointers or transaction objects.

## Ephemeral

The datastore marks a table ephemeral from the module definition (a dedicated definition section, a system table `st_ephemeral_table`, and an in-memory set consulted on every commit). Rows flow through the ordinary transaction, index and subscription machinery. The one difference is at the durability boundary: entries of ephemeral tables are excluded from what the commit log receives, a transaction that touched only ephemeral tables takes no offset, and replay restores the schema but no rows.

## Relay

The relay hub lives in the client-connection layer. It owns channel membership (a grid per channel), the per-connection token bucket, the AOI fan-out and crowd thinning, and the optional bridge that calls a module reducer once per interval through the public `ModuleHost` call path with the database identity as caller. Relay frames are pushed onto a connection's outgoing queue with the existing bounded non-transactional send; they never enter the transaction broadcast queue.

## Responsibilities

- The application decides which workflow runs and which state is durable, ephemeral or presentation-only.
- Configuration decides which processes, operations, channels and bridge reducers may communicate.
- memBUS transports and correlates approved cross-process operations.
- Relay fans out presentation payloads to neighbours within bounds.
- Ephemeral keeps live state transactional and subscribable without disk cost.
- Destination and bridge reducers perform authorization and business validation.
- SpacetimeDB performs reducer execution, transaction commit, durability and subscription work.

## Design boundaries

- same physical Windows machine for memBUS; Ephemeral and Relay are per-process features;
- independent SpacetimeDB processes; no shared database objects, no cross-process transaction;
- explicit routes, allowlists, channels and limits;
- memBUS at least once with reconciliation; relay at most once in order per sender; ephemeral transactional but not durable;
- no arbitrary reducer gateway; no automatic network or coordinator fallback;
- wire compatibility: relay message variants are appended after the upstream ones, and the ephemeral definition section is the last section, so stock clients and modules keep their tags.

Detailed wire layouts, source hooks and internal implementation maps are private engineering material and are not part of this release documentation.
