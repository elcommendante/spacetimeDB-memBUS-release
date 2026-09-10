# Benchmarks

Three campaigns: the **2.10.0-R2 Ephemeral + Relay QUICK campaign** (2026-09-10, re-run on the R2 binaries), the **2.10.0-R2 cross-database comparator** (2026-09-10) and, as history, the **R6 Build 12 memBUS transport checkpoint** (2026-07-16, SpacetimeDB 2.6.1).

## 2.10.0-R2 — Ephemeral + Relay (QUICK tier)

**Classification:** QUICK — one host, one campaign per binary (run 2026-09-10 on the R2 binaries; the R1 numbers of 2026-09-09 are superseded), no repetitions, no warm-up window; regression and development evidence, not a release result. **Host:** 20 logical CPUs, 63 GiB, Windows 11, four unrelated idle database processes in the background. **Boundary:** one benchmark process with N raw v2 WebSocket clients → relay-enabled standalone on loopback → same process; publish → deliver latency on one monotonic clock; reducer round trip = call sent → result received. **Fresh database per group** (a durable flood leaves a durability backlog that contaminates the next case — the first campaign proved it and was discarded).

![Stock vs Ephemeral vs Relay](assets/spacetimedb-membus-2.10.0-r2-stock-vs-ephemeral-vs-relay.png) · interactive: [2.10.0-R2 chart](db-membus-2.10.0-r2-ephemeral-relay-chart.html) (same three-way comparison; the raw rows follow)

### R01 — relay fan-out with AOI (20 Hz per client, 30 s, 12-byte payload)

| Case | Clients | Deliveries/s | Recipients per publish | P50 | P95 | P99 | Server CPU (of 30 s) | Commit-log delta |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 10×10 grid, 2 per cell | 200 | 58,764 | 14.7 | 1.03 ms | 1.72 | 1.97 | 27.0 s (≈0.90 cores) | 150,945 B |
| one cell, all-to-all | 200 | 796,020 | 199 | 1.65 ms | 4.70 | 7.40 | 215.8 s (≈7.2 cores) | 150,945 B |
| 10×10 grid, 10 per cell | 1000 | 1,546,448 | 77.4 | 10.6 ms | 31.2 | 44.1 | 457.5 s (≈15.3 cores) | 755,745 B |

The commit-log delta is exactly the WebSocket connection lifecycle (~755 B per connection); relay traffic adds 0 B. The 1000-client row saturates the host (server 15 cores, benchmark process the rest) and is host-limited, not a relay ceiling. Zero failed clients, zero `RateLimited`, zero other errors.

### R03 — the same traffic through reducers and subscriptions (no AOI)

| Case | Clients | Committed tx/s | Fan-out rows/s | Reducer RTT P50 / P95 / P99 | Server CPU | Commit-log delta (30 s) |
|---|---:|---:|---:|---|---:|---:|
| ephemeral table | 50 | 1,001 | 99,975 | **0.70 / 1.07 / 1.22 ms** | 28.3 s | 37.5 KB (lifecycle) |
| durable table | 50 | 1,000 | 99,926 | 7.04 / 9.32 / 12.2 ms | 15.2 s | 6.19 MB |
| ephemeral table | 200 | 4,000 | 1,598,719 | **2.95 / 6.50 / 8.71 ms** | 409.9 s | 150.9 KB (lifecycle) |
| durable table | 200 | 4,000 | 1,598,678 | 8.71 / 14.3 / 17.7 ms | 125.7 s | 24.74 MB |

Durable: 0.82 MB/s at 4,000 tx/s ≈ 71 GB/day. Ephemeral: nothing beyond lifecycle rows. The ephemeral variant spends more CPU (13.7 vs 4.2 cores at 200 clients): with no durability backlog it delivers each fan-out row as soon as it is committed instead of in the batches the durable path forms while waiting for the commit log; that is the cost of every subscriber receiving every row, and the reason the relay with AOI exists.

### R02 — closed-loop write cost, no subscribers (20 s per point)

| Concurrency | Table | Committed tx/s | RTT P50 / P95 / P99 | Server CPU | Commit-log delta |
|---:|---|---:|---|---:|---:|
| 1 | ephemeral | **16,081** | **0.058 / 0.090 / 0.128 ms** | 23.4 s | 501 B |
| 1 | durable | 285 | 3.33 / 4.34 / 6.17 ms | 2.2 s | 1.17 MB |
| 8 | ephemeral | **96,486** | **0.079 / 0.118 / 0.154 ms** | 99.3 s | 5.8 KB |
| 8 | durable | 1,145 | 7.07 / 8.32 / 11.1 ms | 4.0 s | 4.70 MB |
| 32 | ephemeral | **139,921** | **0.216 / 0.368 / 0.472 ms** | 141.8 s | 23.9 KB |
| 32 | durable | 4,211 | 7.40 / 10.4 / 10.8 ms | 6.6 s | 17.29 MB |

### R1 → R2: the send-path wake-up defect (R02-C1 / R03-200), fixed

The R1 campaign recorded two anomalies: an idle client calling an ephemeral-only reducer saw 8–16 ms round trips (3 ms durable), and at 200 subscribing clients the ephemeral variant had P99 102 ms and 2.7× the durable CPU. Root cause (fork ChangeLog 0.0.0.113): the subscription send worker keeps itself "hot" after a message by sleeping for a short linger window on a Tokio timer and does not listen to its channel during that sleep; Tokio timers tick at 1 ms and the Windows park granularity is up to 15.6 ms, so a reply enqueued during the window waited for the timer. A durable commit masked it because the durability worker keeps the runtime's timer wheel busy. The fix races the linger timer against the channel. Same host, same harness, before (R2 binaries without the fix) → after (R2 binaries):

| Row | Before | After |
|---|---:|---:|
| R02 c=1 ephemeral RTT P50 / P99 | 15.9 / 16.2 ms, 66 tx/s | **0.058 / 0.128 ms, 16,081 tx/s** |
| R02 c=32 ephemeral RTT P50 / P99 | 15.6 / 16.3 ms, 2,341 tx/s | **0.216 / 0.472 ms, 139,921 tx/s** |
| R03 200 clients ephemeral RTT P50 / P99 | 4.14 / 18.8 ms | **2.95 / 8.71 ms** |
| R03 50 clients ephemeral RTT P50 / P99 | 1.24 / 17.2 ms | **0.70 / 1.22 ms** |
| R03 200 clients durable RTT P50 / P99 | 8.71 / 17.3 ms | 8.71 / 17.7 ms (unchanged, as expected) |

Raw rows of both runs: `memBUS/build/2.10.0 - 1/benchmarks/r2-before-linger-fix` and `…/r2-after-linger-fix` in the fork repository.

### Not done

No repetitions, no second host, no RELEASE tier. The relay with the bridge enabled under load and with crowd thinning active were not benchmarked (thinning was above every case here, so the AOI numbers are un-thinned).

## 2.10.0-R2 — cross-database calls (QUICK tier)

**Classification:** QUICK — one host, three repetitions with a fresh alpha/beta pair each, 1 s warm-up + 3 s measure per point (HTTP path: 1 s measure, one point at a time with a TIME_WAIT drain before each). **Boundary:** external HttpClient immediately before `POST /v1/database/alpha/call/<procedure>` → alpha procedure → (memBUS | HTTP to beta) → beta batch reducer commit → commit-aware acknowledgement → response body read. Two-byte payload, `N` logical operations per call in one destination transaction, `C` concurrent closed-loop clients. Harness `memBUS/build/2.10.0 - 1/Invoke-CrossDbComparator.ps1`; run `crossdb-r2-loopback`.

| N × C | memBUS P50 / P99 | memBUS ops/s | HTTP-from-procedure P50 / P99 | HTTP ops/s | noop floor P50 / P99 |
|---|---:|---:|---:|---:|---:|
| 1 × 1 | **0.58 / 0.81 ms** | 1,680 | 1.30 / 2.06 ms | 703 | 0.43 / 0.54 ms |
| 1 × 8 | **0.68 / 0.93 ms** | 11,342 | 1.56 / 2.59 ms | 4,798 | 0.48 / 0.67 ms |
| 1 × 32 | **1.78 / 4.49 ms** | 17,142 | 4.26 / 7.16 ms | 7,340 | 1.25 / 3.63 ms |
| 32 × 1 | **0.95 / 1.20 ms** | 33,021 | 1.76 / 18.6 ms | 15,763 | 0.43 / 0.56 ms |
| 32 × 8 | **2.10 / 4.71 ms** | 108,135 | 2.74 / 4.69 ms | 89,066 | 0.48 / 0.68 ms |
| 32 × 32 | **8.30 / 20.9 ms** | 109,908 | 5,023 / 10,007 ms, 95 failed | 0 | 1.26 / 4.51 ms |

`noop` is the same call path with a procedure that returns immediately (alpha ingress only). Zero memBUS failures in every point. The HTTP path is upstream `procedure_http_request`, which builds a new client and TCP connection per call; it refuses loopback and private addresses in production builds, so the comparator used a benchmark-only standalone built with the upstream feature `allow_loopback_http_for_tests` for all three paths (not shipped). At 32 × 32 that path exhausts the Windows dynamic port range and times out; the row is reported as measured, not as a memBUS win. The transport-only boundary (prepared pre-send → destination dispatch) of the R6 checkpoint was not re-measured; the 2.10.0 harness measures full calls.

## R6 Build 12 — memBUS transport checkpoint (SpacetimeDB 2.6.1)

**Classification:** accepted candidate checkpoint, interactive Windows, same machine; not SCM/Session 0 evidence and not a universal guarantee. Three fresh alpha/beta process pairs, 100 warm-up + 1,000 measured calls per path/pair, two-byte payload, normal Windows scheduling, median of run-level nearest-rank percentiles.

| Boundary | memBUS P50/P95/P99 | Persistent local HTTP P50/P95/P99 | Speedup |
|---|---:|---:|---:|
| prepared pre-send → destination dispatch | `0.0249 / 0.0355 / 0.0455 ms` | `0.1674 / 0.2446 / 0.2942 ms` | `6.72x / 6.89x / 6.47x` |
| prepared pre-send → committed ACK/response | `0.1197 / 0.1826 / 0.2245 ms` | `0.4084 / 0.6036 / 0.7779 ms` | `3.41x / 3.31x / 3.47x` |

All 3,000 operations per path committed. The explicit `<= 0.20 ms P99` target remained open at `0.2245 ms`. The comparator is persistent loopback HTTP, not HTTPS/TLS, and not a memBUS fallback. Charts: [R6 chart](db-membus-benchmark-chart.html), [R1–R6 + 2.10.0-R2 history](db-membus-release-history-chart.html). These numbers belong to the 2.6.1 Build 12 executable and its harness; the 2.10.0-R2 cross-database campaign above is the current same-binary evidence (full-call boundary, different harness — do not compare the two tables row by row).

## Rules for future claims

- same payload, reducer, mutation, auth and commit semantics;
- matched clocks and symmetric boundaries;
- fresh process/state/capability roots, fresh database per group;
- warm-up declared separately;
- raw count and aggregation method retained;
- P50/P95 plus repeated P99/P99.9 where claimed;
- no transport-only multiplier presented as a full-operation result;
- no host-limited row presented as a ceiling;
- no TLS/HTTPS label for the plain loopback HTTP comparator.
