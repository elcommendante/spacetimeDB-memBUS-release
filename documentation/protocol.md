# Protocol and delivery semantics

## Frame v1

- little-endian;
- fixed header: 232 bytes;
- ring records aligned to 8 bytes;
- magic `DBMBUS01`;
- protocol/header version 1;
- final marker `0x44424D425553434D`;
- CRC32C for header and payload.

Important fields include route/channel IDs, endpoints, source database, source PID/epoch, expected destination epoch, sequence, OperationId, correlation ID, schema, payload length, and deadline.

Publication order is copy → Release fence → commit marker → Release tail → `SetEvent`. Consumption Acquire-loads tail, validates the complete record, Release-publishes head, and signals space.

## Route objects

Each directed route owns:

```text
Local\DBMemBus.<route>.map
Local\DBMemBus.<route>.data
Local\DBMemBus.<route>.space
Local\DBMemBus.<route>.shutdown
```

`data` and `space` are auto-reset hints. Correctness always rechecks atomic cursors. `shutdown` is manual-reset.

The control block contains immutable sizing/route fields, producer/consumer claims and decisions, hashes, identities, epochs, state, binding, cursors, and counters. Mutual handshake acceptance is required before traffic.

## Delivery

Contract: at least once, `AUTH=A+`, `ACK=TRANSACTION`, `RESULT=ACK+INBOX`.

| Outcome | Meaning | Action |
|---|---|---|
| `TransactionCommitted` | Destination transaction committed | Mark source outbox acknowledged |
| `AlreadyTransactionCommitted` | Same operation already applied | Treat as acknowledged after inbox verification |
| `Rejected` | Route/auth/schema/input rejection | Correct configuration or request |
| `ExecutionFailed` | Reducer failed; no commit | Correct code/input before retry |
| `BudgetExceeded` | Execution budget exhausted | Retry only under an explicit policy |
| `RetryableFailure` | Typed transient pre-known-commit failure | Bounded same-ID retry |
| `TimedOut` / `Unknown` | Commit state unknown | Reconcile inbox, then same-ID retry if absent |

ACK is emitted after `ReducerOutcome::Committed`. It is not an fsync claim. Exactly once is not claimed.

## Idempotency

OperationId is stable across retry. The destination inbox key plus payload hash ensures:

- identical duplicate: no duplicate effect;
- same ID with different payload: reject;
- new ID: new operation.

## Fail-closed states

Corruption, sequence gap, identity/schema/config mismatch, stale epoch, duplicate ownership, missing peer, shutdown, or irreversible signal failure stops the affected route. No frame is skipped and acknowledged as success.
