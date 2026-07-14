# Security policy

SpacetimeDB-memBUS is designed for trusted, colocated SpacetimeDB processes on one Windows machine. It is not a public network transport.

## Security boundaries

- Windows objects use the `Local\` namespace and an explicit current-user DACL.
- The interactive R6 profile provisions an exact route capability, performs a mutual authenticated handshake and derives per-session frame material.
- Authenticated Frame v2 binds route, session, endpoint/process identity, PID, process epoch, sequence, deadline and key generation with keyed BLAKE3; CRC32C remains corruption detection, not authentication.
- Route, endpoint, database identity, process epoch, schema, sequence, size, marker, deadline, CRC, MAC and replay state are validated before reducer dispatch.
- Topology restricts publishers, subscribers, channels, schemas, and destination reducers.
- The actual source database identity is passed as the destination reducer caller.
- Destination ingress reducers must enforce their own `ctx.sender` allowlist.
- Binary application schema v2 recomputes SHA-256 at the destination and scopes inbox idempotency by source database identity plus operation ID.
- Admission, payload, rate, in-flight, reconciliation and retry state are bounded and fail closed.
- Shared memory never carries live pointers, transactions, Rust references, WASM pointers, or database objects.

## Unsupported exposure

Do not expose named objects to `Everyone`, move them to `Global\`, reuse capability material across unrelated deployments, disable validation, or treat the bundled HTTP listener as an Internet-ready deployment. The R6 evidence covers same-user interactive Windows only; elevated SCM/Session 0/service ACLs remain unaccepted. Actual HTTPS requires an explicitly configured TLS endpoint; the included HTTPS sample refuses downgrade.

## Reporting a vulnerability

Use the hosting repository's private security-advisory channel. Include the affected release, topology shape, Windows version, reproduction, impact, and whether a destination transaction may have committed. Do not publish credentials, tokens, database snapshots, or exploitable object names in a public issue.

Security reports must not be “fixed” by weakening authorization, resetting data, or adding a fallback transport.
