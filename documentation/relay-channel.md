# SpacetimeDB-memBUS-relay — relay channel

The **relay channel** is a non-transactional broadcast lane on the client WebSocket a SpacetimeDB client already holds. A client joins a named channel at an area-of-interest (AOI) position, publishes small opaque payloads, and receives the payloads of its neighbours. Nothing on this lane is a transaction, a table row or a commit-log record. It exists for presentation traffic that must be frequent and cheap — movement, rotation, casts, emotes — while authority stays with reducers.

## Contract

| Property | Relay channel |
|---|---|
| Transport | the existing client WebSocket (v1 and v2 protocols); new message variants appended after the upstream ones, so stock clients keep working and simply never see relay frames |
| Delivery | best effort, **at most once**, in order per sender; a receiver that cannot keep up is disconnected by the normal outgoing-queue policy; nothing is replayed |
| Durability | **0 bytes**: no transaction, no offset, no commit-log record per message (proved per package test) |
| Echo | the sender never receives its own publish |
| Access | per channel: `authenticated` (connection must have a non-anonymous identity) or `anyone` |
| Limits | per process: max payload bytes, max publishes per second per connection (token bucket), max channels per connection; per channel: max subscribers, AOI cell size, crowd threshold, neighbour divisor |
| Membership | server-side grid per channel; a subscriber sees members in its own and neighbouring cells; leaving range emits `Leave`, entering emits `Enter`; disconnect emits `Leave` to neighbours and drops all memberships |
| Bridge (optional) | once per configured interval the server calls a module reducer with the last payload per sender (`Vec<{sender, x, y, payload}>`) using the database identity as caller; the module typically upserts an **ephemeral** table |

## Messages

Client → server: `RelaySubscribe { request_id, channel, x, y }`, `RelayUnsubscribe { request_id, channel }`, `RelayPublish { channel, x, y, payload }`.

Server → client: `RelaySubscribeApplied { request_id, channel, neighbors[] }`, `RelayDeliver { channel, sender, server_micros, payload }`, `RelayEnter { channel, neighbor }`, `RelayLeave { channel, identity }`, `RelayError { request_id, channel, code, message }`.

Error codes: `Disabled`, `UnknownChannel`, `ChannelDenied`, `PayloadTooLarge`, `RateLimited`, `TooManyChannels`, `ChannelFull`, `NotSubscribed`, `AlreadySubscribed`, `InvalidPosition`. A publish rejection carries `request_id = 0`.

`server_micros` is the server's monotonic relay clock. It is a separate clock domain from table timestamps; receivers keep their own estimate of it for render-delay interpolation.

## Configuration (`--membus-relay-config <relay.toml>`)

Required by every relay-enabled standalone; a missing file, an unknown key or a channel-less file fails startup.

```toml
[limits]
max_payload_bytes = 256              # per publish
max_publish_per_second = 30          # per connection, token bucket
max_channels_per_connection = 4

[channels."zone/*"]                  # wildcard: one entry serves zone/<anything>
access = "authenticated"
cell_size = 3000.0                   # AOI grid cell edge, in the game's units
max_subscribers = 2000
crowd_threshold = 60                 # members per cell above which delivery is thinned
neighbor_divisor = 4                 # 1/N of the crowd is delivered when thinning

[channels."zone/hub"]
access = "authenticated"
cell_size = 3000.0
max_subscribers = 2000
crowd_threshold = 60
neighbor_divisor = 4

[channels."zone/hub".bridge]         # optional server-side snapshot into the module
reducer = "relay_ingest"             # must exist in the published module
interval_ms = 1000
```

The package ships this exact file as `config\relay.toml`; `Run-Relay-Test.ps1` asserts the 30/s rate limit, the 256 B bound and the bridge against it.

## Using it from a client

Rust SDK of the fork: `conn.relay().subscribe(channel, x, y)`, `.unsubscribe(channel)`, `.publish(channel, x, y, payload)` plus callbacks `on_relay_deliver / on_relay_enter / on_relay_leave / on_relay_subscribe_applied / on_relay_error`. Generated bindings reach the same API through the `relay()` accessor on `DbConnection` and event contexts.

Any other client (the C#, TypeScript and Unreal SDKs are unchanged upstream code) needs a small extension: encode the three client variants after the upstream `ClientMessage` variants and decode the five server variants before the generic deserializer. The public repository documents the message names and field order above; the demo binary `tools\membus-relay-smoke.exe` is a complete reference implementation of a raw v2 WebSocket relay client (subscribe, publish, deliver, enter/leave, error handling, bridge verification).

## Bridge reducer (server side)

```rust
#[derive(SpacetimeType)]
pub struct RelayIngestRow { pub sender: Identity, pub x: f32, pub y: f32, pub payload: Vec<u8> }

#[spacetimedb::reducer]
pub fn relay_ingest(ctx: &ReducerContext, rows: Vec<RelayIngestRow>) {
    // called once per interval with the last payload per sender; caller == database identity
    for row in rows { /* validate payload, upsert an ephemeral table */ }
}
```

Guard the reducer with `ctx.sender() == ctx.database_identity()`; with the bridge coalescing three publishes into one call per interval the package test reports `bridge-one-row-per-sender PASS` and `bridge-ephemeral-writes-no-commitlog PASS`.

## Measured effect (QUICK tier, 2026-09-09)

| Case (20 Hz per client, 30 s, 12 B payload) | Deliveries/s | Recipients per publish | P50 / P99 | Server CPU |
|---|---:|---:|---|---:|
| 200 clients on a 10×10 AOI grid | 58,744 | 14.7 | 1.04 / 2.21 ms | ~1 core |
| 200 clients in one cell (all-to-all) | 796,040 | 199 | 1.65 / 6.25 ms | 8.7 cores |
| 1000 clients on a 10×10 grid (host-limited) | 1,529,710 | 77 | 20 / 81 ms | 11.9 cores |

Zero failed clients, zero rate-limit rejections, zero other relay errors in every case. The same 200 × 20 Hz through reducers and subscriptions costs 4–12 cores, 4–100 ms and, with a durable table, 71 GB of disk per day. The 1000-client row saturated the host and must not be quoted as a relay ceiling.

## Design notes

- Authority stays with reducers. In the MMORPG the relay carries a 32-byte presentation packet at 20 Hz, the movement-intent reducer still validates allowance and anti-cheat, and a 5-second scheduled reducer persists positions from the bridge's ephemeral table.
- Size AOI cells so a publish reaches tens, not hundreds, of recipients; the all-to-all cell costs eight times the CPU of the grid for the same publish rate.
- Relay frames use uncompressed framing; they are far below the compression threshold.
- Relay logs live under the `membus` log target below the default `ERROR` level; the package sets `membus=debug` in each endpoint's `data\config.toml`.
