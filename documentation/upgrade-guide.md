# Upgrade strategy

SpacetimeDB-memBUS is ported release by release; it does not blindly update the modified tree.

## Layout model

```text
baseline/source/<version>     exact unmodified upstream
baseline/build/<version>-N    baseline proof
memBUS/source/<version>       modified source
memBUS/build/<version>-N      implementation builds/evidence
memBUS/release/<version>-N    packaged release
```

## Port sequence

1. Obtain the exact new SpacetimeDB source as a clean baseline.
2. Verify toolchains and locked release build.
3. Run two unmodified standalone processes on separate ports/data roots.
4. Copy the stable memBUS transport modules deliberately.
5. Re-research the new database/leader/module/reducer APIs.
6. Implement a new narrow version adapter; do not assume v2.6 internals.
7. Reapply the smallest ABI/bootstrap/C# hooks under the feature flag.
8. Build stock and MemBus C# modules and inspect final WASM imports.
9. Run protocol, handshake, adapter, real-process commit, retry/restart, and benchmark gates.
10. Package a new versioned release without overwriting the prior release.

## Upgrade hazards

- procedure/HTTP host-import linker changes;
- `ModuleHost::call_reducer` parameter/outcome changes;
- leader/module lifecycle or cache validity changes;
- C# AOT import allowlists and generated bindings;
- standalone startup/delegate/metrics changes;
- transaction/commit/durability semantics;
- Windows object or Rust atomic layout changes.

Never merge an old modified SpacetimeDB tree wholesale into a newer upstream release.
