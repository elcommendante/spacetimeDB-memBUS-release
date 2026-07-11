# Troubleshooting

## Transfer commits but no `MEMBUS` lines appear

The endpoint `data/config.toml` must include `"membus=debug"` in `[logs].directives`. The clean release seeds include it. A running process does not apply this missing filter immediately, so restart both endpoints after correcting an older extracted data directory. Successful output uses the magenta `MEMBUS` label and includes the source file/line, request publication, destination dispatch and response dispatch.

## Port already in use

The starter fails and does not select another port.

```powershell
Get-NetTCPConnection -State Listen -LocalPort 3910,3920
```

Stop the intended old process or deliberately change the endpoint profile. Do not guess.

## Bundled CLI missing

Restore `2.6.1-1/tools/spacetimedb-cli.exe` from the release and validate the release manifest. The sample does not search `PATH`.

```powershell
& .\2.6.1-1\tools\spacetimedb-cli.exe --version
```

## Release-local key directory

The starters create `2.6.1-1/run/jwt-keys/`; standalone generates the key pair. Use `Start-Demo.ps1` on the first run so beta starts immediately after key generation and within the route handshake timeout. Never commit or distribute the generated private key.

## Principal database missing

The topology identity must exist in the endpoint's restored `data/`. Verify the configured seed archive and hash. If a first-run extraction was interrupted, stop all endpoint processes, remove only the incomplete generated `data/` directory, and restart so the initializer can restore it again.

## Handshake/config/schema/identity mismatch

Compare both `topology.toml` hashes and logical values. Confirm endpoint profile name, database identity, channel schema, reducer allowlist, Windows user, and logon session.

```powershell
Get-FileHash .\2.6.1-1\alpha\topology.toml
Get-FileHash .\2.6.1-1\beta\topology.toml
```

## Stale named object / already exists

Ensure no old endpoint process retains handles. memBUS intentionally refuses to reset an existing mapping. Stop the coordinated pair and restart cleanly; do not delete arbitrary Windows objects while a process may own them.

## CPU affinity failed

Check `[Environment]::ProcessorCount` and use a valid zero-based index. The starter stops the process if affinity cannot be applied.

## `WOULD_BLOCK_TRANSACTION`

Move `ctx.MemBus.Call` outside `WithTx`. Commit source intent first, call, then open a new transaction to record the result.

## `Rejected` or reducer unavailable

Check route target, channel, reducer allowlist, destination `ctx.sender` allowlist, reducer name, schema, and module version. Do not bypass authorization or expose arbitrary reducers.

## `TimedOut` / `Unknown`

Do not report success. Query the destination inbox by OperationId. If absent and retry is approved, reuse the same OperationId and identical payload.

## HTTPS sample rejects local URL

This is intentional. The packaged standalone is plain HTTP. Configure an actual TLS endpoint or use the explicitly named local HTTP sample. There is no downgrade.

## No visible `MEMBUS` log

Confirm the process is the memBUS binary, both `--membus-*` arguments were supplied, topology reached readiness, and the call used the memBUS procedure. Stock modules contain no private import.
