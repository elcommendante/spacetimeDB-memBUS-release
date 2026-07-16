# Delivery and security model

The public contract is intentionally independent of private wire-layout details.

## Delivery

- delivery is at least once;
- each business operation has a stable identity;
- destination processing is idempotent for an exact duplicate;
- conflicting reuse of an operation identity is rejected;
- a response reflects the destination transaction outcome;
- timeout after publication remains uncertain;
- reconciliation queries destination truth before an uncertain mutation is retried.

## Security

- only configured local participants and routes are accepted;
- destination operations are allowlisted;
- local peers authenticate before traffic is accepted;
- message integrity, freshness, size and route context are validated;
- the destination reducer still enforces application authorization;
- security and capacity failures fail closed;
- no credential, capability or player payload is shipped in public documentation.

## Compatibility

Protocol or configuration changes are versioned and require explicit compatibility evidence. There is no silent downgrade and no alternate-transport fallback.

