---
title: "Sandbox.Bind.Proxy"
icon: "🔗"
sources:
  - engine/Sandbox.Bind/Proxy/Proxy.cs
  - engine/Sandbox.Bind/Proxy/PropertyProxy.cs
  - engine/Sandbox.Bind/Proxy/DeepPropertyProxy.cs
  - engine/Sandbox.Bind/Proxy/MethodProxy.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Bind.Proxy

The `Proxy/` subdirectory of `Sandbox.Bind` contains the proxy types that wrap an observed object and dispatch change notifications without forcing every binding consumer to poll. Each `Proxy` exposes a uniform observation surface over a different binding target — a property, a deep property chain, or a method — so the higher-level [Link](../sandbox-bind-links/index.md) layer can subscribe without caring about the underlying shape of the target.

## Proxy Types

* **`Proxy`** — the abstract base. Holds the weak reference to the observed object and exposes the lifecycle hooks `Subscribe` / `Unsubscribe` used by `Link`.
* **`PropertyProxy`** — resolves a single property on a target (e.g. `Player.Health`). When the property's setter fires, the proxy notifies all attached observers.
* **`DeepPropertyProxy`** — resolves a chain of properties (e.g. `Player.Inventory.ActiveWeapon.Ammo`). Internally builds a stack of `PropertyProxy` instances and re-binds intermediate proxies whenever an upstream property changes.
* **`MethodProxy`** — observes the result of a method call. Used for computed bindings where the binding source isn't a field-backed property (e.g. `Player.GetEffectiveSpeed()`).

- **Setter bypass:** Proxies hook into property setters at the IL level. Code that mutates fields directly via reflection or native interop bypasses the proxy and the binding will not refresh.
- **Cycles in deep chains:** A deep property chain that loops back on itself (e.g. `A.B.A.Value`) is rejected at proxy construction; otherwise rebinding would never terminate.
- **Lifetime asymmetry:** A long-lived UI panel binding to a short-lived game object holds the proxy. Because the proxy uses a weak reference, the game object can still be collected — but the proxy itself will linger until the panel is destroyed.

## Related Pages

- [Sandbox.Bind](../sandbox-bind.md) — parent overview of the binding system.
- [Sandbox.Bind.Links](../sandbox-bind-links/index.md) — the consumer of these proxies.
- [Razor UI Compiler Architecture](../../systems/ui/razor/architecture.md)
