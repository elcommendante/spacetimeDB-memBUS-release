# Getting started

## Requirements

- Windows x64;
- at least two logical CPUs for the default affinity layout;
- both endpoint processes under the same Windows user and logon session;
- enough free space to expand approximately 300 MiB of clean endpoint data;
- PowerShell with `Expand-Archive` (included in supported Windows PowerShell versions).

The release includes its matching CLI and generates release-local JWT keys on first start. No global SpacetimeDB installation, pre-existing key directory, or checked-in runtime database directory is required.

The release is not designed for two different machines, Linux, containers with separate Windows sessions, or public Internet exposure.

## Verify the package

From the release root:

```powershell
function Test-Manifest([string]$Directory, [string]$Manifest) {
    foreach ($line in Get-Content (Join-Path $Directory $Manifest)) {
        $expected, $relative = $line -split '  ', 2
        $actual = (Get-FileHash (Join-Path $Directory $relative) -Algorithm SHA256).Hash.ToLowerInvariant()
        if ($actual -ne $expected) { throw "Hash mismatch: $relative" }
    }
}

Test-Manifest .\2.6.1-1 .\RELEASE_MANIFEST.SHA256
Test-Manifest .\2.6.1-1\alpha .\STATIC_MANIFEST.SHA256
Test-Manifest .\2.6.1-1\beta .\STATIC_MANIFEST.SHA256
```

Expected standalone SHA-256:

```text
032DA64B8BEA36733699CC2C654BAE1755D9AD06D7A3615700684C783F2F79AB
```

## Start both endpoints

The first start verifies the SHA-256-pinned seed archives and expands them into ignored local `data/` directories. The recommended launcher coordinates key generation and route startup:

```powershell
Set-Location .\2.6.1-1
.\Start-Demo.ps1
```

It opens two endpoint consoles. Keep them open. Do not wait for the alpha listener before starting beta manually; the listener becomes ready only after both route peers complete the handshake.

Window 1:

```powershell
Set-Location .\2.6.1-1\alpha
.\Start-Alpha.ps1
```

Window 2:

```powershell
Set-Location .\2.6.1-1\beta
.\Start-Beta.ps1
```

The default mapping is alpha `127.0.0.1:3910` on CPU 0 and beta `127.0.0.1:3920` on CPU 1. Starters fail rather than guessing another port, CPU, endpoint, topology, or data directory.

## Verify listeners and affinity

```powershell
Get-NetTCPConnection -State Listen |
    Where-Object { $_.LocalPort -in 3910, 3920 } |
    Sort-Object LocalPort |
    Select-Object LocalAddress, LocalPort, OwningProcess

Get-Process 'spacetimedb-standalone-*' |
    Select-Object Id, ProcessName, ProcessorAffinity, Path
```

Expected masks: alpha `1` (CPU 0), beta `2` (CPU 1).

## Send a committed operation

From the release root or either endpoint folder:

```powershell
.\2.6.1-1\alpha\Send-MemBusSample.ps1
```

Expected:

```text
"TransactionCommitted"
```

## Inspect the destination

```powershell
$beta = 'c2001eade1e3b2db5091092cc766e3ee4290ea1f1187e072f2a040f2d44ab07d'
& .\2.6.1-1\tools\spacetimedb-cli.exe sql --anonymous -s http://127.0.0.1:3920 --no-config $beta `
    'SELECT * FROM membus_effect'
& .\2.6.1-1\tools\spacetimedb-cli.exe sql --anonymous -s http://127.0.0.1:3920 --no-config $beta `
    'SELECT * FROM membus_inbox'
```

## Next reading

- [End-to-end flow](end-to-end.md)
- [Configuration](configuration.md)
- [Commands](commands.md)
- [Troubleshooting](troubleshooting.md)
