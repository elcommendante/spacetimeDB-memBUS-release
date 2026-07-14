# R6 end-to-end operation

## 1. Client trigger

An external client invokes public alpha procedure `membus_send_critical_v2`. This trigger uses the normal SpacetimeDB client endpoint and is outside the measured database-to-database memBUS leg.

Inputs are explicit target endpoint, channel, canonical operation key and bounded byte payload. The procedure parses the operation ID, computes SHA-256 and creates/validates the immutable binary-v2 outbox contract inside a short source transaction.

## 2. Procedure-only host call

After `WithTx` closes, the procedure encodes fixed U128/U256 fields and calls:

```csharp
ctx.MemBus.Call(target, channel, "membus_apply_operation_v2", args, timeout, operationId)
```

Reducers cannot call memBUS. Holding a mutable transaction across the process call is rejected.

## 3. Source admission

The source adapter resolves the exact topology route and deterministic reducer slot. It checks endpoint role, target, channel, allowlist, schema, payload/message limits, request deadline, endpoint/route in-flight limits, token-bucket rate and bounded recovery capacity.

A pre-publication failure is definitely `NotPublished`. It consumes no transport sequence and is never silently queued/dropped.

## 4. Shared-memory request

R6 encodes compact route request v3 inside authenticated Frame v2. The frame binds route/session/process epoch/PID/sequence/deadline, validates with CRC32C plus keyed BLAKE3, and publishes to the alpha->beta SPSC ring using release/acquire atomic ordering. `SetEvent` wakes the beta waiter when required.

## 5. Destination dispatch

Beta wakes, drains the published ring and performs strict validation before one bounded Tokio handoff. The version-sensitive adapter uses a lifecycle-safe validated destination handle and calls the existing `ModuleHost::call_reducer` with the actual alpha database identity as caller.

No direct datastore write or reducer reimplementation exists.

## 6. Reducer and commit

`membus_apply_operation_v2`:

1. validates `ctx.sender` through the destination allowlist;
2. recomputes SHA-256 from payload and fixed-time compares the digest;
3. looks up inbox identity `(operation ID, source database identity)`;
4. returns idempotently for an exact duplicate;
5. rejects the same operation ID/different digest;
6. updates `membus_effect` and inserts `membus_inbox_v2` in one normal transaction.

SpacetimeDB owns reducer execution, transaction creation, commit/durability behavior and subscription publication.

## 7. Commit-aware ACK

Only `ReducerOutcome::Committed` produces `TransactionCommitted`. Beta encodes the result into the authenticated reverse response ring. Alpha validates and correlates the response to the exact attempt, releases admission state and updates the durable source outbox.

The ACK means transaction commit, not fsync. Timeout after publication remains uncertain even if a late ACK subsequently arrives.

## 8. Recovery

Route loss closes the authenticated session. Restart requires a fresh PID/epoch/session handshake; stale frames are rejected. The operation ID and payload digest remain stable across an explicit retry. Reconciliation queries the destination inbox before any uncertain mutation is retried.

## Non-goals

- no shared tables, pointers, WASM memory or transaction objects;
- no cross-database transaction;
- no arbitrary reducer gateway;
- no HTTP/TCP/named-pipe fallback;
- no exactly-once or fsync claim.
