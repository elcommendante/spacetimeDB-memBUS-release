# Protocol and delivery — R6

## Layers

1. **Application operation schema v2:** fixed U128 operation ID, fixed U256 SHA-256 digest and bounded payload.
2. **Compact route request v3:** deterministic reducer slot plus route-owned metadata inside the authenticated transport payload.
3. **Frame v2:** versioned header, route/session identity, epoch/PID/sequence/deadline, lengths, CRC32C and keyed BLAKE3 MAC.
4. **SPSC transport:** one directed request ring and one reverse response ring per route, each with Windows named mapping and event.

R6 changes layers 1-2 and DB-MemBus-owned copy/allocation behavior. It does not weaken Frame v2, ring ordering or destination transaction semantics.

## Publication and validation

The producer builds a complete bounded frame, writes bytes, performs the documented release ordering, publishes the tail and signals the named event. The receiver wakes, acquires the published region, validates marker/length/version/route/session/epoch/PID/sequence/deadline/CRC/MAC, then transfers one owned payload into a bounded nonblocking-first Tokio handoff.

Invalid input never reaches `ModuleHost`. Queue/rate/admission/payload rejection is typed and occurs before publication where possible.

## Authentication

Topology and local principal checks select the exact peer contract. Capability files bootstrap mutual proof and derive per-session frame material. Key ID, generation, session, route, process and sequence are bound. Old, gap, duplicate, exhausted, wrong-route or invalid-MAC traffic fails closed.

Transport authentication complements, not replaces:

- topology reducer allowlist;
- actual source database identity as reducer caller;
- destination `ctx.sender` allowlist;
- source-scoped inbox idempotency;
- destination-recomputed SHA-256.

## Delivery truth

- delivery: at least once;
- acknowledgement: destination transaction outcome;
- result: ACK plus destination inbox;
- timeout: uncertain;
- exactly once: not claimed;
- fsync durability boundary: not claimed.

The stable operation ID remains constant across retries. Every attempt has a distinct correlation ID. A late ACK resolves only its exact attempt/tombstone and cannot satisfy a newer retry.

## Reconciliation

After a published timeout or route loss, bounded uncertain state retains the immutable operation contract. The reconciliation lane queries destination inbox truth without retrying the mutation:

```text
Committed / Failed / Conflict -> terminal evidence
NotFound                      -> same immutable operation may retry
Unknown                       -> remain uncertain
```

The source never turns `Unknown` into success by assumption.
