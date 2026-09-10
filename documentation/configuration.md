# Configuration — 2.10.0-R2 candidate

Two explicit files configure a fork endpoint. Neither has defaults; a missing file, unknown key or inconsistent value fails startup.

## 1. memBUS topology (`--membus-config`, product build only)

`config/topology.template.toml` is immutable package input (topology **version 5**). `Start-Demo.ps1` substitutes only the package-local absolute `capability-directory` and `operator-state-directory`, writing `run/topology.resolved.toml`.

| Field | Package value | Meaning |
|---|---:|---|
| `version` | `5` | topology format; unknown versions fail closed |
| `protocol-version` | `2` | authenticated Frame v2 |
| `queue-capacity-bytes` | `65536` | bounded SPSC ring per direction |
| `max-message-bytes` | `4096` | complete transport message bound |
| `max-operation-payload-bytes` | `3500` | reducer argument bound |
| `overflow` | `reject` | no silent drop |
| `max-in-flight` / `route-max-in-flight` | `128` / `64` | admission bounds |
| `calls-per-second` / `burst-capacity` | `5000` / `128` | route token bucket |
| `request-timeout-ms` / `maximum-request-timeout-ms` | `5000` | topology-owned timeout and ceiling |
| `profile` (channel) | `fast` | source path of the channel: `fast` (no source outbox; a retry must repeat the identical target, channel, operation id and payload) or `full-safety` (durable source outbox + retry/reconcile, two extra source transactions per call); both keep destination authorization, inbox idempotency, normal commit, commit-aware ACK and the explicit Unknown result |
| `payload-contract` | `v2` | binary operation schema |
| `windows.security.profile` | `authenticated-local-v2` | principal, capability, handshake and frame authentication |
| `deployment` | `interactive` | same-user interactive candidate; not Session 0 service proof |

Bundled routes: `alpha -> beta` on channel `alpha-beta` (reducer `membus_apply_operation_v2`, reconciliation `membus_reconcile_operation_v2`) and `alpha-beta-batch` (reducer `membus_apply_batch_v2`). Changing endpoint identity, schema hash, allowlist, security, delivery, acknowledgement or capability ownership is a reviewed change. Do not edit a live resolved topology or add a fallback.

## 2. Relay configuration (`--membus-relay-config`, every relay-enabled build)

```toml
[limits]
max_payload_bytes = 256               # per publish; larger -> RelayError PayloadTooLarge
max_publish_per_second = 30           # token bucket per connection; excess -> RelayError RateLimited
max_channels_per_connection = 4       # more -> RelayError TooManyChannels

[channels."zone/*"]                   # wildcard entry
access = "authenticated"              # or "anyone"
cell_size = 3000.0                    # AOI grid cell edge in application units
max_subscribers = 2000                # -> RelayError ChannelFull
crowd_threshold = 60                  # members per cell above which delivery is thinned
neighbor_divisor = 4                  # fraction delivered while thinning

[channels."zone/hub"]                 # explicit entries win over wildcards
access = "authenticated"
cell_size = 3000.0
max_subscribers = 2000
crowd_threshold = 60
neighbor_divisor = 4

[channels."zone/hub".bridge]          # optional: server-side snapshot into the module
reducer = "relay_ingest"              # must exist in the published module
interval_ms = 1000
```

The package ships this exact file as `config/relay.toml` and its tests assert the limits. A channel a client names that matches no entry is rejected with `UnknownChannel`; an entry with no matching module reducer makes the bridge log an error per interval instead of silently skipping.

## 3. JWT keys (2.10.0 requirement)

`--jwt-pub-key-path` / `--jwt-priv-key-path` are mandatory; the standalone no longer generates keys. The package generates a fresh P-256 pair per demo session with `tools\New-JwtKeyPair.ps1` and deletes it on reset. For your own deployment use the pair your existing tokens were signed with; replacing the key invalidates every issued token.

## 4. Logging

`MemBusDebug = 1` in the endpoint profiles makes the launcher materialise `"membus=debug"` in the data directory's `config.toml` `[logs] directives`. memBUS and relay events (route handshake, publish, deliver, bridge calls) live under that target and are below the default `ERROR` level; `RUST_LOG` alone does not enable them.

## Public configuration contract

- endpoints, routes, channels and bridge reducers are explicit;
- destination operations are allowlisted; relay channels are allowlisted by name or wildcard;
- queues, payloads, concurrency, rate, subscribers and timeouts are bounded;
- missing, malformed or incompatible configuration fails closed;
- overflow is rejected (memBUS) or the slow client is disconnected by the existing WebSocket policy (relay), never silently dropped into a buffer without bound;
- no alternate transport is selected when a route fails;
- named Windows objects are restricted to the configured local principal.

## Endpoint launcher profile

```powershell
@{
    Endpoint         = 'alpha'
    Executable       = 'spacetimedb-standalone-alpha.exe'
    ListenAddress    = '127.0.0.1:3910'
    DataDirectory    = 'data'
    SeedArchive      = '..\seed\alpha-data.zip'
    SeedSha256       = '<accepted-64-hex-sha256>'
    TopologyFile     = '..\run\topology.resolved.toml'
    RelayConfigFile  = '..\config\relay.toml'
    JwtKeyDirectory  = '..\run\jwt-keys'
    CpuIndex         = -1              # normal Windows scheduling
    MemBusDebug      = 1
    DatabaseIdentity = '<exact-64-hex-database-identity>'
}
```

## CPU scheduling

The default package uses normal Windows scheduling. `-CpuIndex` is an explicit experiment; changing it invalidates comparison with published results unless the whole matched campaign is repeated.

## What may be customized

Different local ports, data locations and explicitly approved topology or relay values, only after validating that the complete configuration remains internally consistent. Do not broaden ACLs, remove bounds, or add an HTTP fallback for convenience.
