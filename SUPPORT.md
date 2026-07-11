# Support

Before reporting a problem, read [Troubleshooting](documentation/troubleshooting.md) and collect:

- SpacetimeDB-memBUS release and standalone SHA-256;
- Windows version and logical CPU count;
- endpoint profile and redacted topology;
- listener ports and process affinity masks;
- exact command and typed outcome;
- `MEMBUS` log lines for the affected OperationId;
- whether the destination inbox contains the OperationId;
- whether either process restarted or a module was republished.

Never attach bearer tokens, private keys, complete database data directories, or unredacted production identities to a public issue.

For `TimedOut` or `Unknown`, reconcile the destination inbox before retrying with the same OperationId and payload. Do not generate a new ID blindly.
