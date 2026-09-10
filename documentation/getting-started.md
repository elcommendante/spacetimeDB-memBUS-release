# Getting started with the 2.10.0-R2 candidate

## Requirements

- Windows x64, two or more logical CPUs;
- one writable extracted package directory;
- free loopback ports `3910` and `3920` (and `3930` for the optional relay-only endpoint);
- all endpoint processes under the same interactive Windows user/session;
- Windows PowerShell 5.1 or PowerShell 7;
- approximately 400 MiB free after seed extraction.

The archive is a controlled test candidate. Do not run from inside the ZIP. Unblock it in Windows Properties when required, then extract it completely.

## Verify

```powershell
Set-Location <extract-root>\2.10.0-R2
Set-ExecutionPolicy -Scope Process Bypass
.\Test-PackageIntegrity.ps1
```

Verifies every entry in `SHA256SUMS.txt`. Failure is fatal.

## Start the two-process memBUS demo

```powershell
.\Start-Demo.ps1
```

The launcher checks integrity and free ports, resolves only package-local capability/operator paths into `run\topology.resolved.toml`, provisions the exact topology routes with `membus-security`, **generates a fresh ECDSA P-256 JWT key pair under `run\jwt-keys`** (SpacetimeDB 2.10.0 requires explicit key paths; the package ships no key), restores the accepted alpha/beta seeds, and opens two black consoles. Wait for both to remain open and show magenta `MEMBUS` route-ready lines. Each console also confirms that `data\config.toml` contains `membus=debug`.

## Test memBUS

```powershell
.\Run-MemBus-Test.ps1
```

Three anonymous calls of the public procedure `membus_send_critical_v2` on alpha; each must print `"TransactionCommitted"` and both consoles must show magenta `MEMBUS` runtime lines. Inspect the public fixture on beta with `.\beta\Inspect-Destination.ps1`.

## Test Relay and Ephemeral

```powershell
.\Run-Relay-Test.ps1
```

Publishes the demo module `modules\db_membus_relay_e2e.wasm` on alpha under an anonymous identity and runs `tools\membus-relay-smoke.exe`: three raw WebSocket clients exercise subscribe/AOI, enter/leave, ordered delivery, no echo, the 30/s rate limit, the 256 B payload bound, unknown-channel rejection, the bridge reducer, and prove with the commit-log size that relay traffic and ephemeral writes add 0 bytes. Every assertion prints `CHECK <name> PASS`; the last line must be `RESULT PASS`.

```powershell
.\Run-Ephemeral-Restart-Test.ps1
```

Stops alpha with Ctrl+C, starts it again on the same data directory, and verifies that `durable_marker` replayed one row while the ephemeral `relay_presence_live` is empty. The memBUS route re-establishes after the restart.

## Optional: Ephemeral + Relay without memBUS

```powershell
.\relay-only\Start-RelayOnly.ps1
.\Run-Relay-Test.ps1 -Target relay-only
.\Run-Ephemeral-Restart-Test.ps1 -Target relay-only
```

`relay-only\spacetimedb-standalone-relay-only.exe` is the same fork built without the shared-memory transport: one database process with ephemeral tables and the relay channel, started with only `--membus-relay-config` and a JWT pair.

## Stop and reset

Press `Ctrl+C` in each console, or:

```powershell
.\Stop-Demo.ps1
.\Reset-Demo.ps1
```

Reset removes only generated package children (`alpha\data`, `beta\data`, `relay-only\data`, endpoint `run` folders, root `run` with capabilities and the session JWT keys). It preserves binaries, seeds, templates, documentation and checksums.

## Candidate acceptance record

For the project-lead test of a candidate archive, record:

1. ZIP SHA-256;
2. extraction path and Windows version;
3. integrity result;
4. alpha/beta readiness lines;
5. exact `Run-MemBus-Test.ps1` output;
6. exact `Run-Relay-Test.ps1` CHECK lines and `RESULT`;
7. exact `Run-Ephemeral-Restart-Test.ps1` before/after counts;
8. graceful stop result;
9. reset result and any visible typed error.

The candidate is not accepted solely because it was assembled successfully.
