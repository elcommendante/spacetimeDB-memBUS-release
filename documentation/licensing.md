# Licensing and production deployment

SpacetimeDB-memBUS R2 is based on SpacetimeDB 2.6.1. The upstream work is distributed under the Business Source License 1.1 with an Additional Use Grant and a later change licence.

## Production boundary

The exact SpacetimeDB 2.6.1 Additional Use Grant permits use when an application or service uses no more than **one SpacetimeDB instance in production** and is not used for a Database Service as defined by that licence.

The distinction is important:

- a database is a logical database hosted by SpacetimeDB;
- an instance is a running SpacetimeDB server/process;
- SpacetimeDB-memBUS intentionally connects multiple independent process instances on one machine.

Consequently, the intended multi-process memBUS topology is outside the one-production-instance Additional Use Grant. Before production use, obtain a commercial licence or another applicable arrangement from Clockwork Laboratories. Do not infer production permission from the fact that the binaries run locally or that the project is publicly downloadable.

## Required reading

- bundled root [`LICENSE.txt`](../LICENSE.txt);
- [SpacetimeDB v2.6.1 `LICENSE.txt`](https://github.com/clockworklabs/SpacetimeDB/blob/v2.6.1/LICENSE.txt);
- [upstream SpacetimeDB licensing overview](https://github.com/clockworklabs/SpacetimeDB#license).

The applicable licence can differ by SpacetimeDB version and by subdirectory. Recheck the exact source version before an upgrade or deployment. This page is an operational warning, not legal advice.
