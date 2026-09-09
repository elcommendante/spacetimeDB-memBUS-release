# Delivery and security model

The public contract is intentionally independent of private wire-layout details. Three lanes exist and they must not be confused with each other.

## memBUS (cross-process operations)

- delivery is at least once;
- each business operation has a stable identity;
- destination processing is idempotent for an exact duplicate;
- conflicting reuse of an operation identity is rejected;
- a response reflects the destination transaction outcome (`TransactionCommitted` = committed and subscriptions published; not fsync);
- timeout after publication remains uncertain;
- reconciliation queries destination truth before an uncertain mutation is retried;
- channel profile `fast` drops the source outbox (a retry must repeat the identical operation), `full-safety` keeps it; destination guarantees are the same.

## Ephemeral (tables)

- rows are transactional: a reducer that touches ephemeral and durable rows commits or rolls back atomically;
- rows are subscribable: inserts, updates and deletes reach subscribers exactly like durable rows;
- rows are never durable: no commit-log record, no offset for ephemeral-only transactions, empty after restart;
- the flag is part of the module definition and enforced by the validator (no event or scheduled tables);
- durable → ephemeral migration of an existing table is automatic and clears the rows; ephemeral → durable is rejected.

## Relay (client broadcast lane)

- best effort, at most once, in order per sender; no replay, no buffering without bound (a slow client is disconnected by the existing WebSocket policy);
- the sender never receives its own publish;
- membership and fan-out are server-side per channel with AOI cells and crowd thinning;
- every rejection is a typed `RelayError`; nothing is silently dropped except publishes above the per-connection rate limit, which are reported once per interval as `RateLimited`;
- messages are appended after the upstream protocol variants: a stock client never sees them and keeps working unchanged;
- `server_micros` is a monotonic relay clock, not a table timestamp.

## Security

- only configured local participants and routes are accepted (memBUS);
- destination operations and bridge reducers are allowlisted;
- relay channels are allowlisted by name or wildcard, with `authenticated` or `anyone` access;
- local peers authenticate before memBUS traffic is accepted; message integrity, freshness, size and route context are validated;
- the destination or bridge reducer still enforces application authorization (`ctx.sender()` is the real source identity, or the database identity for bridge calls);
- security and capacity failures fail closed;
- no credential, capability, JWT key or player payload is shipped in the package or the public documentation.

## Compatibility

Protocol or configuration changes are versioned and require explicit compatibility evidence. There is no silent downgrade and no alternate-transport fallback. The relay and ephemeral additions are wire-compatible with stock 2.10.0 clients and modules that do not use them.
