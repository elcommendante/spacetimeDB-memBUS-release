# Getting started

## Requirements

- Windows x64;
- at least two logical CPUs;
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

Test-Manifest .\2.6.1-R2 .\RELEASE_MANIFEST.SHA256
Test-Manifest .\2.6.1-R2\alpha .\STATIC_MANIFEST.SHA256
Test-Manifest .\2.6.1-R2\beta .\STATIC_MANIFEST.SHA256
```

Expected standalone SHA-256:

```text
14F10E81E4B8D9FD2927C1C397CBCEE65FAC6EC057C1B694CE0CB4A0AC4591C7
```

## Start both endpoints

The first start verifies the SHA-256-pinned seed archives and expands them into ignored local `data/` directories. The recommended launcher coordinates key generation and route startup:

```powershell
Set-Location .\2.6.1-R2
.\Start-Demo.ps1
```

It opens two endpoint consoles. Keep them open. Do not wait for the alpha listener before starting beta manually; the listener becomes ready only after both route peers complete the handshake.

Window 1:

```powershell
Set-Location .\2.6.1-R2\alpha
.\Start-Alpha.ps1
```

Window 2:

```powershell
Set-Location .\2.6.1-R2\beta
.\Start-Beta.ps1
```

The default mapping is alpha `127.0.0.1:3910` and beta `127.0.0.1:3920`, both under normal Windows scheduler control. Starters fail rather than guessing another port, endpoint, topology, or data directory. Optional `-CpuIndex` overrides are explicit process-wide affinity experiments and are not the verified R2 default.

## Verify listeners and processes

```powershell
Get-NetTCPConnection -State Listen |
    Where-Object { $_.LocalPort -in 3910, 3920 } |
    Sort-Object LocalPort |
    Select-Object LocalAddress, LocalPort, OwningProcess

Get-Process 'spacetimedb-standalone-*' |
    Select-Object Id, ProcessName, ProcessorAffinity, Path
```

No fixed affinity mask is expected by default.

## Send a committed operation

From the release root or either endpoint folder:

```powershell
.\2.6.1-R2\alpha\Send-MemBusSample.ps1
```

Expected:

```text
"TransactionCommitted"
```

## Inspect the destination

```powershell
$beta = 'c2001eade1e3b2db5091092cc766e3ee4290ea1f1187e072f2a040f2d44ab07d'
& .\2.6.1-R2\tools\spacetimedb-cli.exe sql --anonymous -s http://127.0.0.1:3920 --no-config $beta `
    'SELECT * FROM membus_effect'
& .\2.6.1-R2\tools\spacetimedb-cli.exe sql --anonymous -s http://127.0.0.1:3920 --no-config $beta `
    'SELECT * FROM membus_inbox'
```

## Next reading

- [End-to-end flow](end-to-end.md)
- [Configuration](configuration.md)
- [Commands](commands.md)
- [Troubleshooting](troubleshooting.md)
