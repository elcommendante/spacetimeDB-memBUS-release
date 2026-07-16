# My MMORPG project use case

SpacetimeDB-memBUS is being developed for my MMORPG project's same-machine, multi-database shard.

## Database ownership

| Database | Responsibility | Current state |
|---|---|---|
| `PersistenceDB` | Durable accounts, characters, canonical inventory/equipment, durable progression and authority assignment | Active |
| `GameWorldRegionDB-1` | Live regional presence, movement, combat, AI, loot sources and gameplay projections | Active (`GameWorldDB_1`) |
| `GameWorldRegionDB-2` | Second live region and future world-to-world transfer destination | Scaffold/planned |
| `StatisticsDB` | Server-authored event sink and analytics/ranking facts | Scaffold/event sink |
| `MarketplaceDB` | Future listings, orders, settlement and market history | Planned |

Canonical inventory must not be copied blindly into every region. Persistence owns the durable rows. A GameWorld receives a narrow operation-scoped projection containing only what live gameplay needs, such as equipped item definitions, quantities or charges, combat-relevant stats, equipment revision, authority epoch, and source assignment.

## Example: entering a region

```text
ApiCoordinator creates/reuses EnterWorld OperationId
→ PersistenceDB validates character and durable assignment
→ PersistenceDB stages a thin equipment/stat projection
→ memBUS calls the approved GameWorldRegionDB reducer
→ GameWorld validates source identity, revision and payload hash
→ GameWorld atomically creates live regional state
→ destination transaction commits
→ commit-aware ACK returns
→ ApiCoordinator/Persistence finalize the operation
```

No table or transaction is shared. The projection is serialized command data and the destination creates its own local rows through a normal reducer.

## Example: progression checkpoint

GameWorld owns live progression during an active session. Dirty revisions are coalesced into an immutable checkpoint. memBUS can carry that checkpoint to an approved Persistence reducer, which validates source database, authority epoch, revision and payload hash before durable apply and ACK.

Leave-world and recall must complete the final checkpoint barrier before source cleanup.

## Example: item or loot transfer

```text
source reserves/locks item or loot and commits outbox
→ memBUS sends an OperationId-based command
→ destination checks inbox + payload hash + ownership/content rules
→ destination mutates its own tables and commits
→ ACK returns
→ source finalizes or reconciles
```

The source does not delete canonical state before destination commit. A timeout is uncertainty: inspect the destination inbox, then retry with the same OperationId and payload if absent.

## ApiCoordinator remains the orchestrator

memBUS is not a distributed transaction manager. ApiCoordinator still owns multi-step workflow order, recovery, error mapping, source/destination completion, and StatisticsDB emission.

Target separation:

| Layer | Responsibility |
|---|---|
| ApiCoordinator | Decide and recover the saga/workflow |
| memBUS | Carry approved same-machine commands/events/ACKs with low transport overhead |
| Source reducer/procedure | Validate authority, reserve state, write outbox |
| Destination reducer | Authenticate caller, apply inbox/idempotency, mutate and commit |
| OperationId | Correlate retries and reconciliation across all steps |

The current ApiCoordinator in my MMORPG project uses SDK/HTTP callback paths. Planned integration will route selected high-frequency or latency-sensitive database hops through memBUS while preserving existing operation contracts. It will not silently fall back to the old path when a memBUS route fails.

## Why shared memory helps

For colocated trusted processes, memBUS avoids the public-network and loopback-protocol layers used by a conventional HTTP call. The transport remains bounded, authenticated and local to the Windows host.

Security is narrower because the bus adds no public network listener and local objects are restricted to the configured Windows principal. Explicit topology, operation policy, destination validation, idempotency and the normal SpacetimeDB transaction path remain mandatory.

## Planned topology

See [`MMORPG_MEMBUS_TOPOLOGY.svg`](../MMORPG_MEMBUS_TOPOLOGY.svg). Solid nodes are current project responsibilities; dashed nodes are scaffold/planned. Lines represent logical independent memBUS routes, not one unsafe shared multi-writer queue.

## Performance evidence

The current [R6 benchmark chart](db-membus-benchmark-chart.html), [R1-R6 history](db-membus-release-history-chart.html) and [methodology](benchmarks.md) keep transport-only and full-transaction boundaries separate:

- prepared pre-send to destination dispatch: Build 12 memBUS P50/P95/P99 `0.0249 / 0.0355 / 0.0455 ms`, persistent local HTTP `0.1674 / 0.2446 / 0.2942 ms`;
- prepared pre-send to committed ACK/response: Build 12 memBUS P50/P95/P99 `0.1197 / 0.1826 / 0.2245 ms`, persistent local HTTP `0.4084 / 0.6036 / 0.7779 ms`;
- the matched checkpoint contains three fresh process pairs, 100 warm-up and 1,000 measured calls per path/pair; all 3,000 operations per path committed.

The comparator is plain persistent loopback HTTP, not HTTPS/TLS, and it is never a memBUS fallback. These accepted Build 12 results describe the documented test host and exact benchmark binary; they are not universal guarantees or automatic claims for a differently hashed package rebuild.
