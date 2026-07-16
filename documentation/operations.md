# Operations and lifecycle

## Candidate package lifecycle

The R6 archive and adjacent checksum have passed automated clean-extraction verification and are prepared for controlled project-lead testing. They remain test-candidate assets until the project lead accepts the package and the separately gated six-hour soak is completed.

```text
verify immutable files
-> resolve package-local topology paths
-> provision exact route capability
-> restore accepted clean seeds
-> start alpha and generate package-local JWT keys
-> start beta inside handshake window
-> wait for both listeners and route readiness
-> run/inspect calls
-> Ctrl+C beta then alpha
-> optional reset of generated state
```

The package refuses occupied ports, stale root `run`, checksum mismatch, incomplete data, missing topology route and capability provisioning failure. It does not choose substitutes.

## Route lifecycle

```text
Configured -> Provisioned -> Handshaking -> Ready
Ready -> Reconnecting on peer loss/rotation
Running -> AdmissionClosed -> Draining -> Stopping -> Stopped
```

Peer restart changes PID, epoch and authenticated session. Old frames cannot be replayed into the new process. Backoff is bounded by topology.

## Failure handling

| Situation | Truth |
|---|---|
| payload/args exceed bound | typed not-published rejection |
| queue/rate/in-flight full before publication | typed not-published rejection |
| invalid route/schema/reducer/principal/MAC | fail closed; no reducer entry |
| timeout before publication | definitely not published |
| timeout/route loss after publication | outcome unknown; reconcile |
| reducer failed/budget exceeded | no commit; typed result |
| duplicate same source/ID/digest | idempotent committed duplicate |
| same source/ID/different digest | conflict/failure |

Do not automatically retry a mutation under a new operation ID.

## Process control

Visible candidate acceptance should use the two endpoint consoles and Ctrl+C. `Stop-Demo.ps1` verifies exact PID files and sends Ctrl+C; it fails rather than force-killing after timeout. `Reset-Demo.ps1` refuses live packaged PIDs.

## Security state

Capability files and JWT keys are generated locally under `run`; no package secret exists. Do not copy runtime capability/JWT material into bug reports. Logs must not include payload bodies or player data by default.

The bundled package is an interactive same-user demonstration. Windows service/Session 0 engineering has separate retained acceptance evidence and is not configured or shipped by this demo package.

## Evidence to retain for a failed candidate test

- ZIP/checksum and package integrity output;
- visible typed error from both endpoint consoles;
- exact command and working directory;
- Windows version/user-session mode;
- whether ports were occupied;
- generated non-secret topology hash and process IDs;
- no capability bytes, JWT private key, bearer token or player payload.
