# Development and verification

Source code is not included in this public repository. This page summarizes the quality model used for the binary candidate without publishing private source maps or implementation details.

Every supported SpacetimeDB version begins from a clean, version-specific upstream baseline. memBUS changes are implemented in a separate modified tree and verified with locked dependencies.

Material changes require proof appropriate to their layer:

- focused static checks and tests;
- real Windows cross-process transport where applicable;
- real destination reducer and transaction outcomes;
- restart, authorization, duplicate and uncertainty cases;
- graceful shutdown and state cleanup;
- matched benchmark boundaries for performance claims;
- package integrity and clean-extraction testing for release assets.

Build success alone is not acceptance. Failures remain visible and are not hidden with weaker validation or a fallback transport.

Private engineering documentation contains the exact module ownership, upstream hook ledger, test commands, raw evidence and porting procedure.

