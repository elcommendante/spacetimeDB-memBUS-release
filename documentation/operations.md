# Operations and lifecycle

## Startup

1. Validate endpoint profile and files.
2. Validate the listener and process identity; verify affinity only when an explicit `-CpuIndex` experiment is active.
3. Load/resolve strict topology.
4. Resolve and prewarm the configured principal database.
5. Create/open route mappings and events.
6. Exchange producer/consumer claims and epochs.
7. Advertise readiness only after mutual acceptance.

Start both endpoints within the 15-second sample handshake timeout.

## Runtime

Every inbound route has one blocking waiter. A single Tokio service worker owns route state, pending OperationId correlation, outbound producers, and destination invocation. A burst may coalesce event notifications; cursors remain authoritative.

memBUS logs use the `MEMBUS` level label in magenta and preserve source file/line plus normal message text.

## Shutdown

The shutdown event wakes route waiters. Known pending calls that cannot be completed return unknown/error; they are not converted to success. Current packaged-runtime graceful Ctrl+C proof remains an open acceptance item, so preserve source outbox/inbox reconciliation across restart.

## Restart

A process restart generates a new epoch. Old peers and mappings are stale and fail closed. memBUS does not reset or recreate a mapping silently after an epoch mismatch. Coordinated restart requires old handles to close and both endpoints to complete a new handshake.

## Persistent data

Git does not contain active `data/` directories. It contains two clean, SHA-256-pinned seed archives. On first launch, `Initialize-EndpointData.ps1` validates and restores the matching endpoint data. Subsequent launches reuse that local state. An incomplete existing data directory fails explicitly instead of being overwritten.

To reset the demo deliberately, stop both processes, remove only the two generated endpoint `data/` directories and the release-local `run/` directory, then start alpha followed by beta. The verified seeds will be restored. Never delete runtime data while either process is active.

## Monitoring checklist

```powershell
Get-NetTCPConnection -State Listen |
    Where-Object { $_.LocalPort -in 3910,3920 }

Get-Process 'spacetimedb-standalone-*' |
    Select-Object Id, CPU, ProcessorAffinity, StartTime, Path
```

For an operation, retain its OperationId and inspect source publication, destination dispatch, reducer outcome, response publication, and source correlation. After uncertainty, inspect the destination inbox.

## Capacity

The sample uses a 64 KiB ring, 4 KiB maximum message, 64 maximum in-flight operations, queue overflow rejection, and five-second request timeout. Size from measured payload/concurrency requirements; never increase limits blindly or introduce unbounded queues.

## Backups and packaging

Stop both processes before copying active database data. Release manifests cover the immutable seed archives, binaries, configuration, scripts and documentation; they do not cover changing runtime state or generated private keys.
