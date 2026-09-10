# SpacetimeDB-memBUS-ephemeral — ephemeral tables

An **ephemeral table** is a normal SpacetimeDB table whose rows are never written to the commit log. It is the right shape for state that is *live by definition* — presence, movement segments, animation state, casts in flight, combat events, AI motion, targets — where the durable truth lives elsewhere (a checkpoint table, a persistence database) or is simply recreated after a restart.

## Contract

| Property | Ephemeral table |
|---|---|
| Insert / update / delete / indexes / constraints / SQL | identical to a durable table |
| Transactions | identical: a reducer that touches ephemeral and durable rows commits or rolls back atomically |
| Subscriptions | identical: subscribers receive inserts, updates and deletes |
| Commit log | **never written**; a transaction that touches only ephemeral tables consumes no transaction offset and is never appended; a mixed transaction persists only its durable rows |
| Snapshots / replay | the table's schema is restored, its rows are not: **empty after every restart** |
| Restrictions | cannot be an event table; cannot be a scheduled table; the table must exist in the module |
| Introspection | one row per ephemeral table in the system table `st_ephemeral_table` |

Design rule for module authors: every ephemeral row must be recreated by an existing path (client connect, enter-world, a tick, spawn from content). If no path recreates it, the table must not be ephemeral.

## Declaring

Rust (fork bindings 2.10.0):

```rust
#[spacetimedb::table(accessor = player_movement_segment_live, public, ephemeral)]
pub struct PlayerMovementSegmentLive { #[primary_key] pub character_id: i64, /* ... */ }
```

C# (fork runtime 2.10.0):

```csharp
[SpacetimeDB.Table(Name = "player_movement_segment_live", Public = true, Ephemeral = true)]
public partial struct PlayerMovementSegmentLive { [SpacetimeDB.PrimaryKey] public long CharacterId; /* ... */ }
```

TypeScript modules: the module-definition section is emitted by the shared codegen; the three language schemas are verified equal by the fork's `ensure_same_schema` test.

## Publishing and migration

A new module with ephemeral tables publishes like any module. The validator rejects `ephemeral` on an event table, on a scheduled table, or on a name that does not exist.

**Existing durable table → ephemeral (2.10.0-R2):** add the attribute and publish. The automatic migration plan lists

```text
Changed table player_movement_segment_live becomes ephemeral (existing rows cleared; rows are no longer written to the commit log)
```

The step runs inside the publish transaction: all rows are cleared (they are live state by definition), then the flag is set. The first transaction after the publish is already offset-less. A failed publish rolls both back. `--yes=migrate` (or the interactive confirmation) is required because rows are cleared; `--delete-data` is never needed and must never be used. Old rows remain in historical commit-log segments and are ignored on replay.

**Ephemeral → durable** is rejected (`ChangeTableEphemeralFlag`): there is no durable history to restore; create a new durable table, move consumers, drop the old one.

Real run recorded for 2.10.0-R2: one publish flipped 13 live tables of a game-world database that had replayed 192,024 transactions from a previous version; `st_ephemeral_table` afterwards listed 14 ids.

## Verifying

```sql
SELECT * FROM st_ephemeral_table;         -- one table_id per ephemeral table
SELECT COUNT(*) FROM player_movement_segment_live;   -- 0 right after a restart
```

Commit-log growth: compare the size of the data-directory commit-log segments before and after a burst of ephemeral-only transactions; the delta must be limited to connection lifecycle transactions (~500–750 B per WebSocket connect/disconnect pair). The package's `Run-Relay-Test.ps1` prints exactly this check (`bridge-ephemeral-writes-no-commitlog`), and `Run-Ephemeral-Restart-Test.ps1` proves the restart contract.

## Measured effect (QUICK tier, 2026-09-09)

200 clients writing one position row each at 20 Hz through a reducer, every client subscribed to the table:

| Table kind | Committed tx/s | Fan-out rows/s | Reducer round trip P50 / P99 | Commit log in 30 s |
|---|---:|---:|---|---:|
| durable | 4,000 | 1.6 M | 9.5 / 18.7 ms | 24.7 MB |
| ephemeral | 4,000 | 1.6 M | 4.4 / 101.7 ms | 0.15 MB (lifecycle only) |

Closed-loop write cost with no subscribers: ephemeral needs less server CPU per transaction at every concurrency (0.55 vs 2.59 s at C1, 4.75 vs 9.08 s at C32) and 0 B of disk per transaction versus ~205 B durable.

Cost model (2.10.0-R2 measurements): an idle client calling an ephemeral-only reducer gets its result in about 0.06 ms (16,000 closed-loop calls/s on one connection); at 200 subscribing clients the ephemeral table delivers the same 1.6 M rows/s as the durable one at 3.0 / 8.7 ms P50 / P99 instead of 8.7 / 17.7 ms, but spends about three times the CPU, because every subscriber still receives every row. Use ephemeral tables for high-rate authoritative state and the relay channel with AOI for presentation traffic. The R1 wake-up anomaly (8–16 ms idle calls) was a send-worker timer defect and is fixed in R2.

## Operational notes

- A durable flood (thousands of durable transactions per second with hundreds of subscribers) leaves a durability backlog that can stall the process and inflate its log; ephemeral tables remove that failure mode for live state, they do not fix a module that still floods a durable table.
- Position checkpoints: keep the durable row (for example `player_presence_live`) durable and let a scheduled reducer copy changed positions into it every few seconds; loss on crash is bounded by that interval.
- Ephemeral tables are allowed to be `private` (server-only RAM state such as rate limiters and AI caches).
