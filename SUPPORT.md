# Support

Before reporting a problem, read [Troubleshooting](documentation/troubleshooting.md) and collect:

- SpacetimeDB-memBUS release name and package SHA-256;
- Windows version and logical CPU count;
- the exact package command that failed;
- the complete typed result or error text;
- relevant `MEMBUS` log lines with sensitive values redacted;
- whether either endpoint restarted during the operation;
- whether integrity verification and a clean reset succeed.

Never attach bearer tokens, capability files, private keys, complete database directories or unredacted real database identities to a public issue.

For a timeout or unknown result, do not assume success or failure and do not create an unrelated replacement operation blindly. Preserve the package state and include the observable result in the report.

State clearly that the report concerns the R6 Windows x64 test candidate. Project-lead testing, the approval-only six-hour soak and final release acceptance remain pending.

