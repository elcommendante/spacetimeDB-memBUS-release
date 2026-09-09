# Troubleshooting — 2.10.0-R1 candidate

## Integrity fails

Re-extract the complete ZIP into a new directory and rerun `Test-PackageIntegrity.ps1`. Do not replace one binary or seed independently. Preserve the mismatch output for the candidate report.

## Port already in use

The launcher never chooses another port:

```powershell
Get-NetTCPConnection -State Listen -LocalPort 3910,3920,3930
```

Stop the intended previous process. Do not kill or reconfigure an unrelated listener without confirming ownership.

## Root `run` already exists / JWT key files already exist

The starter refuses ambiguous runtime state and never reuses a key pair. Run `Stop-Demo.ps1`, confirm the standalones exited, then `Reset-Demo.ps1`.

## `Session JWT key is missing`

An endpoint console was started by hand without `Start-Demo.ps1`. Either run the launcher or generate a pair with `tools\New-JwtKeyPair.ps1 -OutputDirectory run\jwt-keys` first. SpacetimeDB 2.10.0 does not create keys on its own.

## Seed missing, incomplete or hash mismatch

The accepted archives are `seed/alpha-data.zip` and `seed/beta-data.zip`. Do not repair generated `data` manually. Stop both endpoints, reset, verify the package and restart.

## Capability provisioning fails

Confirm the package is on a writable local filesystem, `tools/membus-security.exe` passes integrity, alpha/beta/security run as the same Windows user and interactive session, no stale root `run` exists, and the topology template was not edited. The capability directory must not exist before provisioning (the utility creates it with a protected DACL). Do not broaden ACLs to Everyone, copy a capability from another machine, or switch transport.

## One console exits during startup

Read its visible typed error. Typical 2.10.0 causes: missing `--membus-relay-config` (the relay build refuses to start without it), a relay file with no channel, an unknown relay key, missing JWT key paths, database identity or endpoint name not in the resolved topology, incomplete seed, or a listener already owned by another process.

## No magenta `MEMBUS` lines

Confirm each endpoint profile has `MemBusDebug = 1` and startup reported that `data/config.toml` contains `membus=debug`; the standalone builds its filter from that file, not from `RUST_LOG`. Confirm both `--membus-config`/`--membus-endpoint` arguments are present. Listener readiness does not prove route readiness.

## `NotPublished:RouteNotReady`

Keep both endpoint consoles open and wait for their authenticated handshake. Persistent failure indicates peer/config/capability mismatch and must not fall back to HTTP.

## `NotPublished:OperationPayloadTooLarge`

The topology allows 3,500 reducer-argument bytes. Split or redesign the application message deliberately; do not raise the bound without a new review/campaign.

## `Rejected` / execution failure / conflict / `TimedOut` / `Unknown`

Check exact target, channel, reducer allowlist, source database identity, schema, operation ID and payload digest. Do not report an uncertain result as success; preserve the operation ID and immutable payload, inspect the destination inbox and use the approved reconciliation flow before retrying with the same operation ID.

## `WOULD_BLOCK_TRANSACTION`

Move `ctx.MemBus.*` outside `WithTx`. Commit source intent, call asynchronously, then open a new transaction to record the result.

## Relay: `RelayError` codes

| Code | Cause | Fix |
|---|---|---|
| `Disabled` | endpoint started without a relay configuration | add `--membus-relay-config` |
| `UnknownChannel` | no entry or wildcard matches the channel name | add the channel or fix the name |
| `ChannelDenied` | `access = "authenticated"` and the connection is anonymous | authenticate first |
| `PayloadTooLarge` | payload exceeds `max_payload_bytes` | shrink the payload |
| `RateLimited` | more publishes than `max_publish_per_second` | lower the client rate; excess is dropped, not queued |
| `TooManyChannels` / `ChannelFull` | per-connection or per-channel bound | unsubscribe or raise the reviewed bound |
| `NotSubscribed` / `AlreadySubscribed` | state mismatch | subscribe once; `AlreadySubscribed` means the membership already exists |
| `InvalidPosition` | non-finite AOI coordinates | send finite `x, y` |

## Relay: no `RelayDeliver` arrives

The sender never receives its own publish; a second client must be subscribed within AOI range (same or neighbouring cell, `cell_size`). Check `RelaySubscribeApplied.neighbors` and the `RelayEnter` events. A crowded cell above `crowd_threshold` delivers only `1/neighbor_divisor` of the crowd.

## Relay: bridge reducer never called or logs errors

The reducer named in `[channels."<name>".bridge]` must exist in the published module with the argument shape `Vec<{sender, x, y, payload}>`, and the bridge runs only for channels that have at least one publish in the interval. Guard the reducer to accept only the database identity as caller.

## Ephemeral: publish rejected with `ChangeTableEphemeralFlag`

You removed `ephemeral` from an existing ephemeral table. That direction has no durable history to restore: create a new durable table, move consumers, drop the old one.

## Ephemeral: table not empty after restart / rows in the commit log

Check `SELECT * FROM st_ephemeral_table` for the table id. If the id is missing, the attribute did not reach the published module (older bindings, or the table is an event/scheduled table, which the validator rejects). Measure commit-log growth on the WebSocket path only: HTTP `/call` and `/sql` add their own durable connect/disconnect transactions (~500 B each).

## Ephemeral: an idle single call takes 8–16 ms

Known open finding R02-C1 (see Benchmarks). Under load the ephemeral path is faster; a single low-rate ephemeral reducer call should not be expected to beat 3 ms in this build.

## Locked PID/database file while copying

At least one endpoint still runs. Stop both and wait for process exit before reset, delete, copy or archive. Never copy active database state into a release.
