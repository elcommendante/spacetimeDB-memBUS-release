# Troubleshooting — 2.6.1-R6 candidate

## Integrity fails

Re-extract the complete ZIP into a new directory and rerun `Test-PackageIntegrity.ps1`. Do not replace one binary or seed independently. Preserve the mismatch output for the candidate report.

## Port already in use

The launcher never chooses another port:

```powershell
Get-NetTCPConnection -State Listen -LocalPort 3910,3920
```

Stop the intended previous process. Do not kill or reconfigure an unrelated listener without confirming ownership.

## Root `run` already exists

The starter refuses ambiguous security/runtime state. Run `Stop-Demo.ps1`, confirm both standalones exited, then `Reset-Demo.ps1`.

## Seed missing, incomplete or hash mismatch

The accepted archives are `seed/alpha-data.zip` and `seed/beta-data.zip`. Do not repair generated `data` manually. Stop both endpoints, reset, verify the package and restart.

## Capability provisioning fails

Confirm:

- package is extracted to a writable local filesystem;
- `tools/membus-security.exe` passes package integrity;
- alpha/beta/security run as the same Windows user and interactive session;
- no stale root `run` exists;
- topology template was not edited.

Do not broaden ACLs to Everyone, copy a capability from another machine, or switch transport.

## One console exits during startup

Read its visible typed error. Check database identity, endpoint name, resolved topology, seed completeness and listener ownership. The other process may still hold a database lock; stop it before reset/copy.

## No magenta `MEMBUS` lines

The endpoint starters set `RUST_LOG=info,membus=debug`. Confirm the process is the packaged memBUS binary and both `--membus-config`/`--membus-endpoint` arguments are present. Plain listener readiness does not prove route readiness.

## `NotPublished:RouteNotReady`

Keep both endpoint consoles open and wait for their authenticated handshake. Persistent failure indicates peer/config/capability mismatch and must not fall back to HTTP.

## `NotPublished:OperationPayloadTooLarge`

The topology allows 3,500 reducer-argument bytes. Binary-v2 fixed fields consume 52 bytes, leaving exactly 3,448 application bytes. Split or redesign the application message deliberately; do not raise the bound without a new review/campaign.

## `Rejected` / execution failure / conflict

Check exact target, channel, reducer allowlist, source database identity, schema, operation ID and payload digest. The same source/operation ID with different payload must fail.

## `TimedOut` / `Unknown`

Do not report success. Preserve operation ID and immutable payload, inspect destination inbox and invoke the approved reconciliation flow before retry. A retry uses the same operation ID and digest with a new correlation.

## `WOULD_BLOCK_TRANSACTION`

Move `ctx.MemBus.*` outside `WithTx`. Commit source intent, call asynchronously, then open a new transaction to record the result.

## Local HTTP sample rejects the token or URL

It requires one explicit bearer token for the exact approved benchmark identity and accepts only `http://` because it is named persistent local HTTP. The package ships no token. `Send-HttpsSample.ps1` intentionally fails because bundled TLS is not configured.

## Locked PID/database file while copying

At least one endpoint still runs. Stop both and wait for process exit before reset, delete, copy or archive. Never copy active database state into a release.
