# Upgrade policy

SpacetimeDB-memBUS is ported deliberately for each supported SpacetimeDB release.

For a new upstream version, maintainers:

1. obtain and verify a clean official source baseline;
2. build and run the unmodified version;
3. create a separate modified source tree;
4. port the stable memBUS components intentionally;
5. revalidate every version-sensitive integration point;
6. rerun security, cross-process, recovery, transaction and benchmark gates;
7. publish a separate versioned package without overwriting earlier evidence.

The project does not blindly merge a previous modified SpacetimeDB tree into a newer release, silently update dependencies or assume private upstream APIs remain compatible.

