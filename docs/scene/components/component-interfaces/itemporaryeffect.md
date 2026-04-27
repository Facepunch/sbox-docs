---
title: "ITemporaryEffect"
icon: "🕒"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/ITemporaryEffect.cs
  - engine/Sandbox.Engine/Scene/Components/Effects/TemporaryEffect.cs
---

# ITemporaryEffect

`Component.ITemporaryEffect` lets a component report whether its visible effect is *still doing something* and offers a hook for "wind it down now." It's how the [`TemporaryEffect`](../../../systems/effects/temporaryeffect.md) component knows when it's safe to delete the host GameObject — it walks every `ITemporaryEffect` on the object and its children and waits until they all report inactive.

## Quick Working Example

```csharp
using Sandbox;

public sealed class FizzlingSparks : Component, Component.ITemporaryEffect
{
    [Property] public float SparkLifetime { get; set; } = 1.5f;
    private TimeSince _spawnedAt;
    private bool _looping = true;

    protected override void OnEnabled() => _spawnedAt = 0;

    public bool IsActive => _looping && _spawnedAt < SparkLifetime;

    public void DisableLooping()
    {
        // The owning system wants us to die soon. Stop emitting,
        // but let the existing sparks finish naturally.
        _looping = false;
    }
}
```

Pair this on a GameObject with a `TemporaryEffect` component (with `WaitForChildEffects` enabled) and the GameObject will:

1. Live until `_spawnedAt >= SparkLifetime` *and* `_looping == false`,
2. ...then `TemporaryEffect.DestroyAfterSeconds` runs out,
3. ...then the GameObject is destroyed.

## Interface

```csharp
public interface ITemporaryEffect
{
    /// Should return true if the effect is still doing something visible.
    bool IsActive { get; }

    /// Wind down — used to stop looping behaviour so the effect can finish.
    void DisableLooping() { }

    /// Static helper. Walks `go` and its descendants and calls DisableLooping
    /// on every ITemporaryEffect it finds.
    public static void DisableLoopingEffects( GameObject go );
}
```

`IsActive` is **required**. `DisableLooping` has an empty default — implement it only if your effect has a looping mode that needs to be stopped externally.

## When each member matters

| Member | Implement when... | Don't implement when... |
|---|---|---|
| `IsActive` | Always — it's the gate that keeps the GameObject alive while your effect is still visible. | (You always implement it.) |
| `DisableLooping` | The effect can run indefinitely (a continuous emitter, a looping sound) and should respond to "die soon" requests. | The effect already has a fixed lifetime that elapses on its own. |

## Real-world implementations

The engine's own `Decal`, `BeamEffect`, and `ParticleEffect` components all implement `ITemporaryEffect`. Their `IsActive` returns true while particles/beams/decal lifetime is non-zero, and their `DisableLooping` flips `Looped = false` so existing instances finish but no new ones spawn.

When you set up an explosion prefab — `ParticleEffect` + `SoundPointComponent` + `TemporaryEffect` on a single GameObject — the `TemporaryEffect` polls all three; the GameObject only deletes after every effect's `IsActive` is false.

## Static helper: `DisableLoopingEffects`

```csharp
ITemporaryEffect.DisableLoopingEffects( explosionGameObject );
```

Walks the GameObject and all its descendants, calling `DisableLooping()` on every `ITemporaryEffect` it finds. This is what `TemporaryEffect.BecomeOrphan` uses when its parent is destroyed but it wants the orphaned effects to wind down gracefully rather than getting cut off.

- **`IsActive` is polled, not pushed.** `TemporaryEffect.OnUpdate` checks `IsActive` each frame. There's no event you fire when the effect ends — just flip your internal state and `IsActive` will start returning `false` on the next tick.
- **`DisableLooping` may be called multiple times.** Don't allocate or restart anything in it; just set a flag.
- **Disabled components don't count.** `TemporaryEffect.HasActiveEffects` skips disabled components. If you toggle your component off, it's reported as not-active regardless of `IsActive`.
- **Default returns true.** The default `IsActive` (if you forget to implement it) is just whatever your auto-property returns — usually `false`. If your effect doesn't seem to be holding the GameObject alive, double-check that `IsActive` actually returns `true` while it's running.

## See also

- [Temporary Effect (component)](../../../systems/effects/temporaryeffect.md) — the consumer of this interface.
- [Component Interfaces overview](index.md) — the rest of the marker interfaces.
- [Effects](../../../systems/effects/index.md) — particle, beam, decal effects, all of which implement this.
