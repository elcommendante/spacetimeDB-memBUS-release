# Architecture overview

SpacetimeDB-memBUS connects explicitly configured SpacetimeDB processes running on the same Windows machine.

```text
source database process
-> approved local memBUS route
-> destination database process
-> normal destination reducer and transaction
-> committed response or explicit uncertainty
```

Each process keeps its own database, memory, transactions, WASM runtime, durability state and lifecycle. Shared memory is used only to transport bounded message data; it never exposes another process's tables, pointers or transaction objects.

## Responsibilities

- The application decides which business workflow to run.
- Configuration decides which processes and operations may communicate.
- memBUS transports and correlates the approved local operation.
- The destination reducer performs authorization and business validation.
- SpacetimeDB performs reducer execution, transaction commit, durability and subscription work.
- Reconciliation resolves operations whose outcome became uncertain after publication.

## Design boundaries

- same physical Windows machine;
- independent SpacetimeDB processes;
- explicit routes and destination operation allowlists;
- bounded messages and admission;
- at-least-once delivery;
- no arbitrary reducer gateway;
- no shared database objects or cross-process transaction;
- no automatic network or coordinator fallback.

Detailed wire layouts, source hooks and internal implementation maps are private engineering material and are not part of this release documentation.

