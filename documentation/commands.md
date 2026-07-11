# Command reference

Commands assume PowerShell and the release root as the current directory unless stated otherwise.

## Packaged starters

```powershell
.\2.6.1-R2\alpha\Start-Alpha.ps1
.\2.6.1-R2\beta\Start-Beta.ps1
```

Override CPU explicitly:

```powershell
.\2.6.1-R2\alpha\Start-Alpha.ps1 -CpuIndex 2
.\2.6.1-R2\beta\Start-Beta.ps1 -CpuIndex 3
```

The starters validate files, key directory and listener. Normal Windows scheduling is the R2 default; when `-CpuIndex` is supplied explicitly, they also validate the CPU and apply process-wide affinity. They pass:

```text
start
--jwt-key-dir <global-key-directory>
--listen-addr <address:port>
--data-dir <endpoint-data>
--non-interactive
--membus-config <absolute-topology-path>
--membus-endpoint <exact-endpoint-name>
```

`--membus-config` and `--membus-endpoint` are required together.

## Process verification

```powershell
Get-NetTCPConnection -State Listen |
    Where-Object { $_.LocalPort -in 3910,3920 }

Get-Process 'spacetimedb-standalone-*' |
    Select-Object Id, ProcessorAffinity, Path
```

## Packaged samples

```powershell
# Shared-memory call
.\2.6.1-R2\alpha\Send-MemBusSample.ps1

# Bundled local listener: explicitly plain HTTP
.\2.6.1-R2\alpha\Send-LocalHttpSample.ps1 `
    -BearerToken '<approved-benchmark-token>'

# Actual TLS endpoint: refuses http://
.\2.6.1-R2\alpha\Send-HttpsSample.ps1 `
    -ServerUrl 'https://your-endpoint.example' `
    -BearerToken '<approved-benchmark-token>'
```

The v2.6.1 standalone has no native TLS certificate start option. HTTPS requires an explicit TLS terminator/reverse proxy.

## Direct CLI example

For an automated smoke run without visible endpoint consoles:

```powershell
.\2.6.1-R2\Start-Demo.ps1 -Headless
```

```powershell
$alpha = 'c200c612cf22f9e2b7881c82cf55edd0734b1c95137c1e2616863dcf3672d926'
$operation = [guid]::NewGuid().ToString('N')

& .\2.6.1-R2\tools\spacetimedb-cli.exe call --anonymous -s http://127.0.0.1:3910 --no-config `
    $alpha membus_send_critical beta alpha-beta $operation '[71,79]'
```

Retry an uncertain operation with the same ID and payload:

```powershell
& .\2.6.1-R2\tools\spacetimedb-cli.exe call --anonymous -s http://127.0.0.1:3910 --no-config `
    $alpha membus_retry_pending $operation beta alpha-beta '[71,79]'
```

## Inspect state

```powershell
$beta = 'c2001eade1e3b2db5091092cc766e3ee4290ea1f1187e072f2a040f2d44ab07d'

& .\2.6.1-R2\tools\spacetimedb-cli.exe sql --anonymous -s http://127.0.0.1:3920 --no-config `
    $beta 'SELECT * FROM membus_effect'

& .\2.6.1-R2\tools\spacetimedb-cli.exe sql --anonymous -s http://127.0.0.1:3920 --no-config `
    $beta "SELECT * FROM membus_inbox WHERE operation_id = '$operation'"
```

## C# API

```csharp
Result<MemBusResponse, MemBusError> result = ctx.MemBus.Call(
    targetEndpoint: "beta",
    channel: "alpha-beta",
    reducer: "membus_apply_operation",
    payload: encodedArgs,
    timeout: TimeSpan.FromSeconds(5),
    operationId: operationId
);
```

This API is procedure-only and must be outside `WithTx`.

## Demonstration functions

| Name | Kind | Summary |
|---|---|---|
| `membus_send_critical` | Procedure | Write source outbox, call beta, write returned outcome |
| `membus_retry_pending` | Procedure | Replay the same operation/payload after uncertainty |
| `membus_apply_operation` | Reducer | Check source identity and apply inbox/effect idempotently |
| `membus_https_benchmark_apply` | Reducer | Identity-restricted network comparison using the same helper |

## Typed outcomes

`TransactionCommitted`, `AlreadyTransactionCommitted`, `Rejected`, `ExecutionFailed`, `BudgetExceeded`, `RetryableFailure`, `TimedOut`, and `Unknown`.

## Source-build commands

From the complete SpacetimeDB-memBUS source tree, not the binary-only release:

```powershell
cargo fmt --all -- --check
cargo check --locked -p db-membus -p db-membus-spacetimedb
cargo test --locked -p db-membus
cargo test --locked -p db-membus-spacetimedb --test service
cargo clippy --locked -p db-membus -p db-membus-spacetimedb --all-targets --no-deps -- -D warnings
cargo build --locked --release -p spacetimedb-standalone --features db-membus
dotnet build modules\db-membus-e2e-cs\db-membus-e2e-cs.csproj -t:Rebuild -c Release -p:DB_MEMBUS=1
```

Topology/transport probe binaries are developer artifacts and are not included in `2.6.1-R2`.
