# Complete end-to-end flow

This page follows one `membus_send_critical` call from the source procedure to the committed destination effect and back to the source ACK.

## 1. Durable source intent

The source C# procedure accepts an explicit 32-hex OperationId and payload. Inside a short local `WithTx`, it inserts or validates a `membus_outbox` row with state `Pending` and a stable payload fingerprint. The transaction closes before any cross-process wait.

## 2. Procedure-only API

Outside `WithTx`, the procedure calls:

```csharp
ctx.MemBus.Call(
    "beta",
    "alpha-beta",
    "membus_apply_operation",
    encodedArgs,
    TimeSpan.FromSeconds(5),
    operationId
);
```

The C# runtime encodes `DBMCALL1`, version, target, channel, reducer, timeout, OperationId, and BSATN reducer arguments. It invokes `db_membus_0.1::procedure_membus_call`.

Reducers and views use a trapping synchronous stub. Calling while a mutable transaction is open returns `WOULD_BLOCK_TRANSACTION`.

## 3. Source service validation

The async host import delegates to the process `MemBusService`. The service decodes the envelope and verifies:

- caller database equals the configured local principal;
- request target/channel has an outbound request route;
- reducer appears in the channel allowlist;
- OperationId is not already in flight;
- peer epoch and route state are valid.

It creates a request frame with the authenticated route metadata and deadline.

## 4. Frame construction

Protocol v1 encodes a 232-byte little-endian header plus payload and zero padding. It includes route/channel IDs, endpoint IDs, source database identity, process ID/epoch, expected destination epoch, sequence, OperationId, correlation ID, schema, deadline, payload CRC32C, header CRC32C, and a final commit marker.

## 5. SPSC publication

The source producer:

1. reads ring head/tail and checks bounded capacity;
2. copies header, payload, and padding with the marker cleared;
3. executes a Release fence;
4. writes the final commit marker;
5. Release-publishes the new tail cursor;
6. signals the named `data` event with Win32 `SetEvent`.

Queue full produces explicit backpressure. Critical frames are not overwritten or dropped.

## 6. Windows wake and destination decode

A destination-owned blocking waiter is sleeping in `WaitForMultipleObjects` on the route's `data` and `shutdown` events. Windows schedules it after the signal. The waiter Acquire-loads the ring state and validates marker, sizes, protocol, route, endpoint/database identities, epochs, schema, sequence, CRCs, and deadline.

After Release-publishing the new head and signalling space, the waiter sends the decoded frame through `blocking_send` into a bounded Tokio MPSC channel. The waiter never invokes reducers or mutates service state.

## 7. Tokio dispatch and destination invocation

The single service worker receives the frame, validates it against the authenticated route, decodes the request envelope, and repeats the destination reducer allowlist check.

The v2.6 adapter currently performs:

```text
get_database_by_identity
→ leader
→ host.module()
→ ModuleHost::call_reducer
```

The actual source database identity becomes the reducer caller. No owner identity or client identity is fabricated.

## 8. Normal reducer transaction

`membus_apply_operation` is public because cross-database caller identity cannot invoke a private reducer. It enforces `ctx.sender` against a destination-owned allowed-source table.

The reducer then:

1. checks `membus_inbox` by OperationId;
2. rejects reuse with a different payload hash;
3. returns safely for an identical duplicate;
4. increments the example effect;
5. inserts the inbox result;
6. returns through the normal transaction executor.

Only `ReducerOutcome::Committed` becomes `TransactionCommitted`.

## 9. Commit-aware ACK

After the destination invocation returns, the service encodes `DBMRSP01` with the typed outcome and same OperationId. It selects the reverse response route and publishes another checksummed SPSC frame using the same atomic/event protocol.

`TransactionCommitted` proves the destination transaction committed. It does not prove an fsync boundary.

## 10. Source response correlation

The source response waiter wakes, validates and decodes the frame, and sends it to the service worker. The worker matches OperationId to the pending oneshot and returns the encoded response to the host import and C# procedure.

The source procedure opens a new short local transaction and updates its outbox state to the typed outcome.

## 11. Timeout and reconciliation

If the source deadline expires before the response arrives, the result is `TimedOut`/unknown. The source must query/reconcile the destination inbox. A retry uses the same OperationId and payload. A new ID would risk applying a duplicate business effect.

## Full path summary

```text
source outbox commit
→ C# procedure-only MemBus call
→ private async ABI
→ source validation
→ checksummed request frame
→ SPSC Release publication
→ SetEvent / Windows scheduling
→ destination Acquire validation
→ Tokio dispatch
→ database → leader → module resolution
→ ModuleHost::call_reducer
→ destination authorization + inbox idempotency
→ normal transaction commit
→ checksummed response frame
→ reverse SPSC + SetEvent
→ source validation/correlation
→ typed ACK
→ source outbox outcome commit
```

The accepted performance timer excludes the source outbox transactions and CLI startup. See [Benchmarks](benchmarks.md).
