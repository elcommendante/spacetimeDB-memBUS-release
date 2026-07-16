# Operation lifecycle

## Normal operation

1. A client invokes an approved source procedure.
2. The procedure records or validates its operation identity and calls memBUS after closing any mutable source transaction.
3. memBUS validates the configured local route, participant and bounds.
4. The destination process accepts the operation only if its local security and route contract match.
5. The destination invokes the approved reducer through the normal SpacetimeDB execution path.
6. SpacetimeDB completes or rejects the destination transaction.
7. memBUS correlates the result with the source operation.

The external client trigger is not the local database-to-database transport itself.

## Result truth

- committed: the destination transaction reported committed;
- rejected/failed: the destination did not report a committed transaction;
- not published: the operation did not enter the transport;
- timed out/unknown: publication may have occurred and reconciliation is required.

A committed result is not an fsync claim. Timeout is never converted into success, and a retry must preserve the same business operation identity.

## Explicit non-goals

- arbitrary reducer execution;
- shared tables, pointers, WASM memory or transaction objects;
- distributed ACID across databases;
- exactly-once delivery claim;
- automatic HTTP/TCP/named-pipe fallback.

