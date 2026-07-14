# Licensing and production deployment

SpacetimeDB-memBUS 2.6.1-R6 is based on SpacetimeDB 2.6.1. The upstream work is distributed under the Business Source License 1.1 with an Additional Use Grant and later change licence.

## Production boundary

The SpacetimeDB 2.6.1 Additional Use Grant permits use when an application or service uses no more than **one SpacetimeDB instance in production** and is not used for a Database Service as defined by that licence.

SpacetimeDB-memBUS intentionally connects multiple independent server processes on one machine. The intended production topology is therefore outside that one-production-instance grant unless another right applies. Before production use, obtain a commercial licence or another applicable written arrangement from Clockwork Laboratories. Local execution or public download does not grant additional production rights.

## Required reading

- bundled [`LICENSE.txt`](../LICENSE.txt);
- [SpacetimeDB v2.6.1 `LICENSE.txt`](https://github.com/clockworklabs/SpacetimeDB/blob/v2.6.1/LICENSE.txt);
- upstream version-specific licensing material.

Licensing can differ by source version and subdirectory. Recheck the exact version before upgrade/deployment. This page is an operational warning, not legal advice.
