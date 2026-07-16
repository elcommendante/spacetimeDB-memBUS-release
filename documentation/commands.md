# Command reference — 2.6.1-R6 candidate

Run these commands from PowerShell in the extracted package directory. The package is a controlled Windows x64 test candidate, not a production release.

## Verify package integrity

```powershell
.\Test-PackageIntegrity.ps1
```

Checks every packaged file against `SHA256SUMS.txt`. A missing, modified or unexpected file fails with a non-zero exit status.

## Start the two endpoints

```powershell
.\Start-Demo.ps1
```

Starts two independent local SpacetimeDB processes on the documented demo ports and creates package-local runtime state. Wait until both consoles report `MEMBUS ... runtime ready`.

For automated local evaluation, `-Headless` is available. Visible consoles are recommended for manual candidate acceptance.

## Run the memBUS sample

```powershell
.\Run-MemBus-Test.ps1
```

Executes the bundled approved database-to-database operation and requires a committed destination result. It returns non-zero for rejection, timeout, unavailable routes or any other non-committed outcome.

## Inspect the destination

```powershell
.\beta\Inspect-Destination.ps1
```

Performs a read-only inspection of the demo destination so the committed operation can be confirmed independently.

## Stop and reset

```powershell
.\Stop-Demo.ps1
.\Reset-Demo.ps1
```

`Stop-Demo.ps1` requests graceful shutdown and waits for the known package processes. `Reset-Demo.ps1` refuses to run while they are live and removes only generated package-local state.

## Optional comparator sample

```powershell
.\alpha\Send-LocalHttpSample.ps1 -BearerToken '<approved-identity-token>'
```

Runs the bundled persistent loopback HTTP comparator. No token ships in the package, and HTTP is never used as a memBUS fallback.

## Advanced utilities

The package includes a security utility used by the launcher to provision its fixed demo routes. Its advanced lifecycle commands are intended for controlled evaluation only; use `\.\tools\membus-security.exe --help` and do not expose generated capability material.

The exact private ABI, operation schema, host adapter and reducer implementation are intentionally documented only in the private development repository.

