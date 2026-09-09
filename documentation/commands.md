# Command reference — 2.10.0-R1 candidate

Run these commands from PowerShell in the extracted package directory. The package is a controlled Windows x64 test candidate, not a production release.

## Package scripts

| Command | Purpose |
|---|---|
| `.\Test-PackageIntegrity.ps1` | checks every packaged file against `SHA256SUMS.txt`; a missing, modified or unexpected file fails |
| `.\Start-Demo.ps1 [-Headless]` | integrity, free-port check, `run\topology.resolved.toml`, route capabilities, **session JWT key pair**, seed restore, alpha (3910) and beta (3920) consoles |
| `.\Run-MemBus-Test.ps1 [-Calls 3]` | anonymous calls of `membus_send_critical_v2` on alpha; each must return `TransactionCommitted` |
| `.\beta\Inspect-Destination.ps1` | anonymous read of the public fixture table on beta |
| `.\Run-Relay-Test.ps1 [-Target alpha\|relay-only] [-DatabaseName relaydemo]` | publishes the relay/ephemeral demo module and runs the raw-WebSocket smoke client; every `CHECK` must PASS |
| `.\Run-Ephemeral-Restart-Test.ps1 [-Target alpha\|relay-only]` | graceful restart of the endpoint; durable row replayed, ephemeral table empty |
| `.\relay-only\Start-RelayOnly.ps1 [-ListenAddress 127.0.0.1:3930]` | single-process Ephemeral + Relay endpoint without memBUS |
| `.\Stop-Demo.ps1` | Ctrl+C to relay-only, beta, alpha (exact PID files); never force-kills |
| `.\Reset-Demo.ps1` | refuses live PIDs; removes generated data, `run` folders, capabilities and session JWT keys only |
| `.\alpha\Start-Alpha.ps1 [-CpuIndex n]`, `.\beta\Start-Beta.ps1` | endpoint consoles used by the launcher; `-CpuIndex` is an explicit experiment |
| `.\tools\New-JwtKeyPair.ps1 -OutputDirectory <dir>` | generates the P-256 pair the 2.10.0 standalone requires (`id_ecdsa`, `id_ecdsa.pub`) |

## Standalone flags added by the fork

```text
spacetimedb-standalone.exe start
    --listen-addr 127.0.0.1:3910 --data-dir <dir>
    --jwt-pub-key-path <id_ecdsa.pub> --jwt-priv-key-path <id_ecdsa>   # 2.10.0 requires explicit key paths
    --non-interactive
    --membus-config <topology.toml> --membus-endpoint <name>            # memBUS (product build only)
    --membus-relay-config <relay.toml>                                   # Relay (required in every relay-enabled build)
```

`spacetimedb-standalone-relay-only.exe` accepts the same flags minus the two memBUS ones. A missing relay file, an unknown key or a channel-less relay file fails startup; there is no default relay configuration.

## Security utility

```powershell
.\tools\membus-security.exe list-routes --topology <resolved.toml> --endpoint alpha
.\tools\membus-security.exe provision-route --topology <resolved.toml> --endpoint <owner> --capability-directory <dir> --route-id <id> --not-before-utc-ms <ms> --not-after-utc-ms <ms>
.\tools\membus-security.exe --help
```

The launcher runs exactly these for the bundled routes. Do not expose generated capability material.

## SpacetimeDB CLI (stock 2.10.0)

```powershell
.\tools\spacetimedb-cli.exe call --anonymous -s http://127.0.0.1:3910 --no-config <db> membus_send_critical_v2 beta alpha-beta <operation-id> '[71,79]'
.\tools\spacetimedb-cli.exe sql  --anonymous -s http://127.0.0.1:3910 --no-config relaydemo 'SELECT * FROM st_ephemeral_table'
.\tools\spacetimedb-cli.exe publish <name> --server http://127.0.0.1:3910 --module-path <module> --delete-data=never --yes=remote,migrate,break-clients,skip-login
```

A publish that turns an existing durable table ephemeral prints `Changed table <name> becomes ephemeral (existing rows cleared; …)` in its migration plan and needs the `migrate` confirmation. Never use `--delete-data`.

## Module declarations

```rust
#[spacetimedb::table(accessor = presence_live, public, ephemeral)]   // Rust
```

```csharp
[SpacetimeDB.Table(Name = "presence_live", Public = true, Ephemeral = true)]   // C#
```

```csharp
var result = ctx.MemBus.Call(target: configuredTarget, channel: configuredChannel, reducer: approvedOperation,
                             payload: serializedPayload, timeout: deadline, operationId: stableOperationId);   // memBUS, procedure-only
```

## Relay client operations

| Operation | Fields | Answer |
|---|---|---|
| `RelaySubscribe` | `request_id, channel, x, y` | `RelaySubscribeApplied { request_id, channel, neighbors }` or `RelayError` |
| `RelayUnsubscribe` | `request_id, channel` | silent success or `RelayError` |
| `RelayPublish` | `channel, x, y, payload` | fire-and-forget; rejection arrives as `RelayError { request_id: 0 }` |
| server → client | `RelayDeliver { channel, sender, server_micros, payload }`, `RelayEnter { channel, neighbor }`, `RelayLeave { channel, identity }` | — |

The fork's Rust SDK exposes them as `conn.relay().subscribe/unsubscribe/publish` and `on_relay_*` callbacks; `tools\membus-relay-smoke.exe` is a complete raw-WebSocket reference client (`membus-relay-smoke.exe <http-base> <wasm> <db-name> <data-dir>`).

The exact private ABI, host adapter and reducer implementation are intentionally documented only in the private development repository.
