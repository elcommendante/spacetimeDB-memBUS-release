# Upgrade policy

SpacetimeDB-memBUS is ported deliberately for each supported SpacetimeDB release.

For a new upstream version, maintainers:

1. obtain and verify a clean official source baseline;
2. build and run the unmodified version;
3. create a separate modified source tree;
4. port the stable components (memBUS transport, Ephemeral, Relay) intentionally;
5. revalidate every version-sensitive integration point against the upstream-file ledger;
6. rerun security, cross-process, recovery, transaction, relay/ephemeral and benchmark gates;
7. publish a separate versioned package without overwriting earlier evidence.

The project does not blindly merge a previous modified SpacetimeDB tree into a newer release, silently update dependencies or assume private upstream APIs remain compatible.

## Ports so far

| Package | Upstream | What changed for operators |
|---|---|---|
| `2.6.1-1` … `2.6.1-R6` | SpacetimeDB 2.6.1 | memBUS transport R1–R6 |
| (private) | SpacetimeDB 2.8.3 | lane diagnosis, `fast` / `full-safety` channel profiles, database-to-database comparator |
| `2.10.0-R2` | SpacetimeDB 2.10.0 | topology v5; **JWT key paths are mandatory** (`--jwt-pub-key-path`, `--jwt-priv-key-path`); **Ephemeral** tables; **Relay** channel with mandatory `--membus-relay-config`; relay-only build |

## Upgrading a database from a stock 2.6.1 / 2.8.3 standalone to 2.10.0-R2

1. Stop the old process. Copy the data directory; keep the original untouched as rollback.
2. Start the fork standalone on the copy with the **same JWT key pair** the old CLI used (`%LOCALAPPDATA%\SpacetimeDB\config\id_ecdsa[.pub]` for a stock install) so existing identities and tokens stay valid, plus a relay configuration file. The first start replays the commit log and rewrites `metadata.toml` to the new version; a 192,024-transaction game-world database replayed without intervention in the reference migration.
3. Rebuild modules against the fork bindings, mark the live tables `ephemeral`, add bridge reducers, publish with `--delete-data=never --yes=migrate,...` from the owner identity. The plan lists every `becomes ephemeral` step.
4. Keep the old process directory as rollback until acceptance. Never run old and new on the same ports at the same time.

Only the operator's own tooling migrates data; the fork never replaces or resets a data directory.
