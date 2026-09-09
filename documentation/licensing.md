# Licensing and production deployment

SpacetimeDB-memBUS 2.10.0-R1 is based on SpacetimeDB 2.10.0. The upstream work is distributed under the Business Source License 1.1 with an Additional Use Grant and a later change licence (change date 2031-09-03 for 2.10.0).

## Production boundary

The SpacetimeDB 2.10.0 Additional Use Grant permits use when an application or service uses no more than **one SpacetimeDB instance in production** and is not used for a Database Service as defined by that licence.

SpacetimeDB-memBUS (the transport) intentionally connects multiple independent server processes on one machine. That topology is outside the one-production-instance grant unless another right applies. SpacetimeDB-memBUS-ephemeral and SpacetimeDB-memBUS-relay are per-process features; running them on a single production instance does not by itself leave the grant, but the rest of your deployment must still comply. Before production use, obtain a commercial licence or another applicable written arrangement from Clockwork Laboratories. Local execution or public download does not grant additional production rights.

## Required reading

- bundled [`LICENSE.txt`](../LICENSE.txt) (SpacetimeDB 2.10.0 text);
- [SpacetimeDB v2.10.0 `LICENSE.txt`](https://github.com/clockworklabs/SpacetimeDB/blob/v2.10.0/LICENSE.txt);
- upstream version-specific licensing material.

Licensing can differ by source version and subdirectory. Recheck the exact version before upgrade/deployment. This page is an operational warning, not legal advice.
