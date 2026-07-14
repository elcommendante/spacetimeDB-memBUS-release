# Command reference — 2.6.1-R6 candidate

Commands assume PowerShell in the extracted `2.6.1-R6` package unless noted.

## Package integrity

```powershell
.\Test-PackageIntegrity.ps1
```

Prerequisite: immutable `SHA256SUMS.txt`. Side effect: none. Output: `PASS` plus verified count. Any missing/malformed/mismatched path throws and returns non-zero.

## Coordinated startup

```powershell
.\Start-Demo.ps1 [-Headless]
```

Prerequisites: ports 3910/3920 free, no root `run`, writable package, same interactive Windows user. Side effects: creates package-local runtime topology, capability files, JWT keys, expanded endpoint data and PID/state files; opens two PowerShell endpoint processes. It provisions only topology-owned routes and never selects a fallback.

`-Headless` hides endpoint consoles but retains the same behavior; visible mode is recommended for candidate acceptance.

## Endpoint startup

Normally invoked by `Start-Demo.ps1`:

```powershell
.\alpha\Start-Alpha.ps1 [-CpuIndex <0..62>]
.\beta\Start-Beta.ps1  [-CpuIndex <0..62>]
```

`-1` (default) uses normal Windows scheduling. A non-negative value is an explicit process-affinity experiment and fails if invalid/refused.

Each starter passes:

```text
start --jwt-key-dir <package-run> --listen-addr <explicit> --data-dir <endpoint-data>
      --non-interactive --membus-config <resolved-topology> --membus-endpoint <exact-name>
```

## Shared-memory sample

```powershell
.\Run-MemBus-Test.ps1
.\alpha\Send-MemBusSample.ps1 [-AlphaServer <http://...>] [-AlphaIdentity <64hex>] [-Payload <byte[]>]
```

The wrapper requires `TransactionCommitted`; another result throws. The sample generates one operation ID and triggers `membus_send_critical_v2 beta alpha-beta <id> <payload>` through the normal alpha client endpoint. The subsequent database-to-database hop uses memBUS only.

## Destination inspection

```powershell
.\beta\Inspect-Destination.ps1
```

Runs read-only SQL for `membus_effect` and `membus_inbox_v2` against the fixed beta identity. Failure is non-zero.

## Persistent local HTTP comparator sample

```powershell
.\alpha\Send-LocalHttpSample.ps1 -BearerToken '<approved-identity-token>'
```

An explicit one-line bearer token is mandatory. The script computes exact U128/U256 JSON, calls `membus_https_benchmark_apply_v2`, and accepts only `http://` loopback by design. No token ships in the package.

```powershell
.\alpha\Send-HttpsSample.ps1 -ServerUrl 'https://...' -BearerToken '...'
```

The bundled standalone has no TLS configuration; this script intentionally fails instead of pretending that local HTTP is HTTPS or creating a fallback.

## Stop and reset

```powershell
.\Stop-Demo.ps1 [-TimeoutSeconds <1..120>]
.\Reset-Demo.ps1 [-WhatIf]
```

Stop reads exact endpoint PID files, verifies the process name, sends Ctrl+C and waits. It does not force-kill after timeout. Reset refuses while a packaged PID is live, then removes only generated package-local state.

## Security utility

```powershell
.\tools\membus-security.exe list-routes `
  --topology <absolute-topology> --endpoint <name>

.\tools\membus-security.exe provision-route `
  --topology <absolute-topology> --endpoint <source> `
  --capability-directory <absolute-create-new-directory> `
  --route-id <exact-hex-route-id> `
  --not-before-utc-ms <epoch-ms> --not-after-utc-ms <epoch-ms>
```

`list-routes` is read-only and prints no secret. `provision-route` creates one route capability and refuses overwrite. Other lifecycle commands are `inspect-route`, `stage-rotation`, `activate-key`, `retire-key` and `verify-store`; inspect the package utility's `--help` and public operations documentation before use. Never print capability bytes.

## C# surface

```csharp
ctx.MemBus.Call(target, channel, reducer, payload, timeout, operationId);
ctx.MemBus.Reconcile(target, channel, operationId, expectedPayloadSha256, timeout);
ctx.MemBus.FanOut(channel, reducer, payload, timeout, operationId);
ctx.MemBus.ReconcileFanOut(channel, operationId, expectedPayloadSha256, timeout);
```

These APIs are procedure-only, use a private async host import and must not be called while holding a mutable `WithTx` transaction.

## Reference v2 operations

| Operation | Kind | Purpose |
|---|---|---|
| `membus_send_critical_v2` | procedure | one bounded durable outbox attempt |
| `membus_send_critical_policy_v2` | procedure | explicit immutable deadline/attempt policy |
| `membus_retry_pending_v2` | procedure | retry same operation and digest only |
| `membus_reconcile_critical_v2` | procedure | reconcile uncertain destination inbox truth |
| `membus_fanout_critical_v2` | procedure | topology-owned independent target calls |
| `membus_reconcile_fanout_v2` | procedure | independent target reconciliation |
| `membus_many_to_one_send_v2` | procedure | one approved publisher to fixed target contract |
| `membus_apply_operation_v2` | reducer | sender AUTH, SHA-256, inbox and effect mutation |
| `membus_https_benchmark_apply_v2` | reducer | approved-identity local HTTP comparator |

Typed results include committed, rejected, execution/budget failure, retryable failure, not-published reasons and published-outcome uncertainty. Timeout is not success.
