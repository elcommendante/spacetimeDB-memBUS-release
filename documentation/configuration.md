# Configuration — R6

The candidate uses topology format v4 and authenticated protocol v2. Values are explicit; missing or incompatible configuration fails closed.

## Package topology

`config/topology.template.toml` contains fixed endpoint identities and route policy. `Start-Demo.ps1` substitutes only:

- `capability-directory` -> package-local `run/capabilities`;
- `operator-state-directory` -> package-local `run/operator-state`.

The resolved topology is written to `run/topology.resolved.toml`. It is runtime state and must not be copied back over the template.

## Bounds

| Setting | Candidate value | Behavior |
|---|---:|---|
| `queue-capacity-bytes` | 65,536 | bounded request/response SPSC rings |
| `max-message-bytes` | 4,096 | entire transport message bound |
| `max-operation-payload-bytes` | 3,500 | reducer args bound; v2 fixed fields leave 3,448 app bytes |
| `max-in-flight` | 128 | endpoint admission ceiling |
| `max-reconciliation-in-flight` | 8 | lane reserved for uncertainty resolution |
| `route-max-in-flight` | 64 | route admission ceiling |
| `calls-per-second` | 5,000 | route rate limit |
| `burst-capacity` | 128 | bounded token-bucket burst |
| `maximum-request-timeout-ms` | 30,000 | topology-owned timeout ceiling |
| `maximum-fanout-destinations` | 3 | bounded topology-owned fanout target count |

Overflow rejects; it never silently drops or switches transports.

## Authenticated local profile

```toml
[windows]
namespace = "local"
deployment = "interactive"

[windows.security]
profile = "authenticated-local-v2"
algorithm = "blake3-keyed-256"
capability-owner = { mode = "current-user-development" }
operator-reader = { mode = "current-user-development" }
```

This is the accepted interactive same-user profile. It is not Windows service/Session 0 proof. Never broaden Windows object ACLs to Everyone.

## Bundled route

```text
alpha (publisher database identity c2000a...e4b23)
-> channel alpha-beta
-> beta (subscriber database identity c20027...9190)
-> schema hash 076f77...cf6e70
-> allowlisted reducer membus_apply_operation_v2
-> reconciliation procedure membus_reconcile_operation_v2
```

Route policy is a contract. Changes to identity, publisher/subscriber set, schema, reducer, acknowledgement, delivery, capability owner or bounds require review and new acceptance evidence.

## CPU configuration

Default `CpuIndex = -1` uses normal Windows scheduling, which produced accepted R6 numbers. Whole-process pinning is an explicit experiment only and its results are not comparable unless the complete matched campaign is rerun.
