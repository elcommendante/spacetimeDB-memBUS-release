# Benchmarks

## Current R6 matched campaign

**Classification:** candidate, interactive Windows, same machine. It is not SCM/Session 0 evidence and not a universal guarantee.

Exact boundaries:

- transport: prepared pre-send QPC immediately before memBUS frame construction or HTTP `SendAsync` -> destination dispatch QPC;
- full: the same prepared pre-send QPC -> correlated committed ACK/HTTP response dispatch at the source.

Both paths execute the same binary-v2 destination mutation: approved sender, SHA-256 verification, source-scoped inbox idempotency, one effect update and a normal SpacetimeDB transaction commit.

Campaign:

1. five fresh alpha/beta process pairs;
2. fresh expanded seeds and newly provisioned authenticated route per pair;
3. normal Windows scheduling across 20 logical CPUs;
4. 100 warm-up plus 1,000 measured operations per path/pair;
5. two-byte application payload;
6. every operation required a committed result;
7. report the median of the five run-level nearest-rank P50/P95 values; do not pool samples.

| Boundary | R6 memBUS P50/P95 | Persistent local HTTP P50/P95 | Speedup P50/P95 |
|---|---:|---:|---:|
| transport -> dispatch | `0.0250 / 0.0382 ms` | `0.1304 / 0.1755 ms` | `5.22x / 4.59x` |
| full commit -> ACK/response | `0.2762 / 0.4615 ms` | `0.4731 / 0.6884 ms` | `1.71x / 1.49x` |

All 5,000 memBUS and 5,000 HTTP headline calls committed. Host-load samples were 0-10%.

Payload scaling retained a full-path win at 256 B and exact configured maximum 3448 B. A 3449-byte application payload was rejected before publication as `NotPublished:OperationPayloadTooLarge`.

## Tail diagnostic

One additional fresh pair executed 10,000 measured calls/path after warm-up:

| Path | Full P50 | P95 | P99 | P99.9 | Max |
|---|---:|---:|---:|---:|---:|
| R6 memBUS | `0.2525` | `0.3533` | `0.5829` | `1.1249` | `13.6173` ms |
| persistent local HTTP | `0.4524` | `0.6480` | `0.8375` | `1.0497` | `2.9818` ms |

This is a diagnostic, not a repeated tail acceptance campaign. R6 wins through P99; one memBUS outlier raises max and its P99.9 is slightly worse.

## History chart caveat

The release-history chart plots R2-R6 on the prepared pre-send -> committed ACK boundary. R1 used an older asymmetric diagnostic timer and is retained as a clipped, explicitly non-comparable historical bar. The single HTTP bar on the history panel is the current R6 same-session comparator; it must not be used as if it were measured in every historical session.

Open:

- [current R6 chart](db-membus-benchmark-chart.html);
- [R1-R6 history chart](db-membus-release-history-chart.html).

## Rules for future claims

- same payload, reducer, mutation, auth and commit semantics;
- matched clocks and symmetric boundaries;
- fresh process/state/capability roots;
- warm-up declared separately;
- raw count and aggregation method retained;
- P50/P95 plus repeated P99/P99.9 where claimed;
- no transport-only multiplier presented as a full-operation result;
- no TLS/HTTPS label for the current plain loopback HTTP comparator.
