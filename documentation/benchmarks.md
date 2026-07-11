# Benchmarks

## Accepted boundaries

Payload: two bytes (`GO`), same Windows host, warm state, one excluded warm-up plus 50 measured samples.

| Metric | Min | Mean | P50 | P95 | Max |
|---|---:|---:|---:|---:|---:|
| MemBus publication → destination dispatch | 0.026 ms | 0.0406 ms | 0.040 ms | 0.055 ms | 0.061 ms |
| Persistent HTTP send → host dispatch | 0.304 ms | 0.763 ms | 0.722 ms | 1.271 ms | 1.466 ms |
| Full MemBus publication → committed ACK at source | 0.710 ms | 3.570 ms | 3.670 ms | 4.662 ms | 5.220 ms |
| Full HTTP send → mutation/commit → response | 1.139 ms | 3.152 ms | 2.761 ms | 6.432 ms | 6.801 ms |

## Interpretation

- MemBus is 18.05x faster at P50 for transport-to-dispatch.
- HTTP is 1.33x faster at P50 for the full committed operation.
- MemBus is 1.38x faster at P95 for the full committed operation.

These are not contradictory: destination database/leader/module resolution and normal reducer/commit work dominate the full MemBus median.

The local HTTP test used `http://`, not TLS. Do not label it HTTPS.

## ApiCoordinator context

The live PersistenceDB → ApiCoordinator → GameWorldDB chain measured about 31.186 ms P50 from source committed callback to target committed callback and 62.153 ms P50 for the external HTTP roundtrip. Its callback loop included a 10 ms polling delay, so it is not a pure network comparison.

## Rules for new results

- same payload, reducer, mutation and commit semantics;
- separate transport, resolution, reducer/commit, ACK, and client/CLI time;
- report P50/P95/P99/max and raw sample count;
- exclude and identify warm-up/cold start explicitly;
- do not compare different clocks as a sub-millisecond one-way timer;
- retain raw evidence and exact commands;
- never claim a full-operation multiplier from a transport-only timer.

## Planned optimization

The next iteration will time database lookup, leader lookup, module lookup, reducer/commit, ACK publication, and response transit separately, then evaluate a lifecycle-safe destination cache. It must preserve authorization, normal reducer execution, commit-aware ACK, and uncertainty.
