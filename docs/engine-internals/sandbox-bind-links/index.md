---
title: "Sandbox.Bind.Links"
icon: "🔗"
sources:
  - engine/Sandbox.Bind/Links/Link.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Bind.Links

The `Links/` subdirectory of `Sandbox.Bind` contains `Link` — the connector between an observed binding source (a [Proxy](../sandbox-bind-proxy/index.md)) and an observer (typically a Razor panel that needs to invalidate its render output).

Where a `Proxy` answers "what changed?", a `Link` answers "what should happen when it changes?".

## Lifecycle

1. **Resolve** — given a binding expression (`Player.Inventory.ActiveWeapon.Ammo`), the link resolves the appropriate proxy type from `Sandbox.Bind.Proxy`.
2. **Activate** — the link subscribes to the proxy. From this moment forward, mutations on the source flow into the observer.
3. **Notify** — when the proxy raises a change, the link invokes its callback. The Razor panel typically responds by marking itself dirty and scheduling a rebuild.
4. **Dispose** — when the binding scope ends, the link unsubscribes and releases its references.

- **Re-entrancy:** A link callback that triggers another mutation on the same source can cause an infinite re-notification loop. Links suppress redundant notifications within a single change cycle.
- **Target destruction:** When the proxy detects that its weak target reference is dead, it raises a final notification and the link disposes itself; the observer is responsible for handling a "binding ended" signal.
- **Hotload reset:** During C# hotload, all live `Link` instances must be torn down before the assembly is unloaded. The binding system listens for the hotload event and disposes its link registry first.

## Related Pages

- [Sandbox.Bind](../sandbox-bind.md) — parent overview of the binding system.
- [Sandbox.Bind.Proxy](../sandbox-bind-proxy/index.md) — the upstream proxies that links subscribe to.
- [Razor UI Compiler Architecture](../../systems/ui/razor/architecture.md)
