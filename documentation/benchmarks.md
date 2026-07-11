# Benchmarks

## R2 accepted boundaries

Payload: two bytes (`GO`), same Windows host, warm state, 100 warm-up calls plus 1,000 measured calls per path. Benchmark debug observation was disabled and both processes used normal Windows scheduling.

| Metric | P50 | P95 |
|---|---:|---:|
| memBUS symmetric pre-send QPC -> destination dispatch QPC | 0.0206 ms | 0.0348 ms |
| Persistent local HTTP symmetric pre-send QPC -> destination dispatch QPC | 0.2885 ms | 0.3738 ms |
| Full memBUS pre-send -> committed ACK correlation at source | 0.2931 ms | 0.4301 ms |
| Full HTTP pre-send -> committed response correlation | 0.6558 ms | 0.8629 ms |

## Interpretation

- memBUS is 14.00x faster at P50 and 10.74x faster at P95 for transport-to-dispatch.
- memBUS is 2.24x faster at P50 and 2.01x faster at P95 for the full committed operation.
- All 1,000 measured operations per path committed.
- The result was repeated across three fresh process runs and confirmed against the final R2 binary.

R2 removes repeated destination resolution from the normal hot path with a prewarmed, generation-safe cache. Every call still validates the current identity, leader and module generation and still enters the normal `ModuleHost`, reducer, transaction and commit path.

The local HTTP test used `http://`, not TLS. Do not label it HTTPS.

These figures describe the documented test host and are not a performance guarantee for every machine. Debug-level per-operation observation materially increases latency; the accepted comparison therefore keeps the benchmark observer disabled for both paths.

Open the [printable R2 chart](db-membus-benchmark-chart.html).

## ApiCoordinator context

The earlier live PersistenceDB -> ApiCoordinator -> GameWorldDB chain measured about 31.186 ms P50 from source committed callback to target committed callback and 62.153 ms P50 for the external HTTP roundtrip. Its callback loop included a 10 ms polling delay, so it is not a pure network comparison and is not mixed into the matched R2 chart.

## Rules for new results

- use the same payload, reducer, mutation and commit semantics;
- separate transport, resolution, reducer/commit, ACK, and client/CLI time;
- report P50/P95/P99/max and raw sample count;
- exclude and identify warm-up/cold start explicitly;
- do not compare different clocks as a sub-millisecond one-way timer;
- retain raw evidence and exact commands;
- never claim a full-operation multiplier from a transport-only timer.

## Remaining optimization area

The shared-memory handoff is no longer the dominant full-call cost. The largest measured phase is normal reducer execution and transaction work. Future work must preserve authorization, destination idempotency, normal reducer execution, commit-aware ACK and uncertainty semantics.
