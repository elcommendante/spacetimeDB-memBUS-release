# Security policy

SpacetimeDB-memBUS is intended for trusted, colocated SpacetimeDB processes on one Windows machine. It is not a public network transport.

## Security boundaries

- participants, routes and destination operations are explicitly configured;
- local peers authenticate before messages are accepted;
- message integrity, freshness, size and route context are validated;
- the actual source identity remains available to destination authorization;
- destination reducers must enforce their own application access rules;
- admission, payload, retry and reconciliation state are bounded;
- shared memory carries message data only, never live database objects or pointers;
- invalid or incompatible input fails closed;
- no automatic HTTP, TCP, named-pipe or coordinator fallback exists.

Do not broaden local object permissions, reuse security material across unrelated deployments, disable validation or expose the included demo as an Internet-ready service.

## Reporting a vulnerability

Use the hosting repository's private security-advisory channel. Include the affected release, Windows version, reproduction and impact. Do not publish credentials, keys, database snapshots, private object names or player data in a public issue.

Security issues must not be “fixed” by weakening authorization or introducing a fallback transport.
