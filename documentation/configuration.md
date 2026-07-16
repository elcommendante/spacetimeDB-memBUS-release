# Configuration — R6 candidate

The demonstration uses one explicit, versioned topology template. Startup resolves package-local runtime paths into `run/topology.resolved.toml`; generated runtime files must not replace the template.

## Public configuration contract

- endpoints and directed routes are explicit;
- destination operations are allowlisted;
- queues, payloads, concurrency, rate and timeout are bounded;
- missing, malformed or incompatible configuration fails closed;
- overflow is rejected rather than silently dropped;
- no alternate transport is selected when a route fails;
- named Windows objects are restricted to the configured local principal;
- the demo exposes no general arbitrary-reducer gateway.

The exact protocol layout, cryptographic construction, internal operation schema and private route identities are intentionally not published.

## CPU scheduling

The default package uses normal Windows scheduling. Optional CPU selection is an experimental evaluation control; changing it invalidates direct comparison with published benchmark results unless the complete matched campaign is repeated.

## What may be customized

The test candidate may be evaluated with different local ports, data locations and explicitly approved topology values only after validating that the complete configuration remains internally consistent. Do not broaden ACLs, remove bounds or add an HTTP fallback for convenience.

