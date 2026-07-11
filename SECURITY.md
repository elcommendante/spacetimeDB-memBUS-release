# Security policy

SpacetimeDB-memBUS is designed for trusted, colocated SpacetimeDB processes on one Windows machine. It is not a public network transport.

## Security boundaries

- Windows objects use the `Local\` namespace and an explicit current-user DACL.
- Route, endpoint, database identity, process epoch, schema, sequence, size, marker, and CRC fields are validated.
- Topology restricts publishers, subscribers, channels, schemas, and destination reducers.
- The actual source database identity is passed as the destination reducer caller.
- Destination ingress reducers must enforce their own `ctx.sender` allowlist.
- Shared memory never carries live pointers, transactions, Rust references, WASM pointers, or database objects.

## Unsupported exposure

Do not expose named objects to `Everyone`, move them to `Global\`, disable validation, or treat the bundled HTTP listener as an Internet-ready deployment. Actual HTTPS requires an explicitly configured TLS endpoint; the included HTTPS sample refuses downgrade.

## Reporting a vulnerability

Use the hosting repository's private security-advisory channel. Include the affected release, topology shape, Windows version, reproduction, impact, and whether a destination transaction may have committed. Do not publish credentials, tokens, database snapshots, or exploitable object names in a public issue.

Security reports must not be “fixed” by weakening authorization, resetting data, or adding a fallback transport.
