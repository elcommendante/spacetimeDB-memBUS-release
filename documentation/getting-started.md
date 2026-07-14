# Getting started with the 2.6.1-R6 candidate

## Requirements

- Windows x64, two or more logical CPUs;
- one writable extracted package directory;
- free loopback ports `3910` and `3920`;
- both endpoint processes under the same interactive Windows user/session;
- approximately 500 MiB free after seed extraction.

The R6 archive is not published yet. Once the project-lead acceptance test is complete and the asset is available, do not run from inside the ZIP. Unblock it in Windows Properties when required, then extract it completely.

## Verify

```powershell
Set-Location <extract-root>\2.6.1-R6
Set-ExecutionPolicy -Scope Process Bypass
.\Test-PackageIntegrity.ps1
```

The command verifies every entry in `SHA256SUMS.txt`. Failure is fatal.

## Start

```powershell
.\Start-Demo.ps1
```

The launcher resolves only package-local capability/operator paths, provisions the exact topology route, validates and expands accepted seed archives, generates package-local JWT keys and opens separate alpha/beta consoles. It does not invent a port, endpoint, identity, route or fallback.

Wait for both consoles to remain open and show a magenta `MEMBUS` readiness message.

## Test

```powershell
.\Run-MemBus-Test.ps1
```

Expected result:

```text
"TransactionCommitted"
```

Inspect the public effect and private v2 inbox:

```powershell
.\beta\Inspect-Destination.ps1
```

## Stop and reset

Press `Ctrl+C` in beta and then alpha, or run:

```powershell
.\Stop-Demo.ps1
```

After both exact processes exit:

```powershell
.\Reset-Demo.ps1
```

Reset removes only generated package children (`alpha/data`, `beta/data`, endpoint PID folders and root `run`). It preserves binaries, seeds, topology template, documentation and checksums.

## Candidate acceptance record

For the project-lead test of a locally generated candidate archive, record:

1. ZIP SHA-256;
2. extraction path and Windows version;
3. integrity result;
4. alpha/beta readiness;
5. exact `Run-MemBus-Test.ps1` output;
6. destination effect/inbox observation;
7. graceful stop result;
8. reset result and any visible typed error.

The candidate is not accepted solely because it was assembled successfully.
