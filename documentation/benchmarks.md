# Benchmarks

## Accepted Build 12 matched checkpoint

**Classification:** candidate, interactive Windows, same machine. It is not SCM/Session 0 evidence and not a universal guarantee.

Exact boundaries:

- transport: prepared pre-send QPC immediately before memBUS frame construction or HTTP `SendAsync` -> destination dispatch QPC;
- full: the same prepared pre-send QPC -> correlated committed ACK/HTTP response dispatch at the source.

Both paths execute the same binary-v2 destination mutation: approved sender, SHA-256 verification, source-scoped inbox idempotency, one effect update and a normal SpacetimeDB transaction commit.

Campaign:

1. three fresh alpha/beta process pairs;
2. fresh expanded seeds and newly provisioned authenticated route per pair;
3. normal Windows scheduling across 20 logical CPUs;
4. 100 warm-up plus 1,000 measured operations per path/pair;
5. two-byte application payload;
6. every operation required a committed result;
7. report the median of the three run-level nearest-rank P50/P95/P99 values; do not pool samples.

| Boundary | memBUS P50/P95/P99 | Persistent local HTTP P50/P95/P99 | Speedup P50/P95/P99 |
|---|---:|---:|---:|
| transport -> dispatch | `0.0249 / 0.0355 / 0.0455 ms` | `0.1674 / 0.2446 / 0.2942 ms` | `6.72x / 6.89x / 6.47x` |
| full commit -> ACK/response | `0.1197 / 0.1826 / 0.2245 ms` | `0.4084 / 0.6036 / 0.7779 ms` | `3.41x / 3.31x / 3.47x` |

All 3,000 memBUS and 3,000 HTTP headline calls committed. Host-load samples were recorded at 43-55% and were not normalized. The full-path P50 target was exceeded; the explicit `<= 0.20 ms P99` target remains open at `0.2245 ms`.

The accepted campaign executable hash is pinned in private evidence, but that executable file was not retained. The downloadable test candidate is a fresh current-source rebuild with a separate hash and has passed its own clean-extraction functional test. This page does not transfer Build 12 performance acceptance to a differently hashed binary or claim identical performance for the packaged rebuild.

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
