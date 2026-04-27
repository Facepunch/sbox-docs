---
title: "IPressable"
icon: "👆"
created: 2026-04-25
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/IPressable.cs
---

# IPressable

`Component.IPressable` makes a component interactable — a button, a door, a pickup, a sittable chair, a lever. The interface defines the **press lifecycle** (Hover → Press → Pressing → Release) plus tooltip support so the player can see what an interaction does before triggering it.

`PlayerController` and similar interaction systems trace into the world, find an `IPressable`, and drive its lifecycle in response to player input (typically the **Use** key).

## Quick Working Example

```csharp
using Sandbox;

public sealed class InteractiveButton : Component, Component.IPressable
{
    [Property] public string Label { get; set; } = "Press";
    public event Action OnPressed;

    public bool Press( IPressable.Event e )
    {
        OnPressed?.Invoke();
        return true; // success — Release will be called when the player lets go
    }

    public void Release( IPressable.Event e )
    {
        // Cleanup if Press allocated anything
    }

    public IPressable.Tooltip? GetTooltip( IPressable.Event e )
    {
        return new IPressable.Tooltip(
            Title:       Label,
            Icon:        "touch_app",
            Description: "Tap to activate" );
    }
}
```

Drop this on a GameObject with a Collider. Now when a `PlayerController` looks at it and presses **Use**, the button fires once. The tooltip ("Press · Tap to activate") appears whenever the player aims at it.

## Interface

```csharp
public interface IPressable
{
    public record struct Event( Component Source, Ray? Ray = default );
    public record struct Tooltip( string Title, string Icon, string Description,
                                  bool Enabled = true, IPressable Pressable = default );

    void  Hover( Event e )    { }   // Player started looking at this
    void  Look( Event e )     { }   // Player is still looking — every frame
    void  Blur( Event e )     { }   // Player stopped looking

    bool  Press( Event e );          // Required — return true on success
    bool  Pressing( Event e ) => true; // Per frame; return false to cancel
    void  Release( Event e )  { }   // Press finished

    bool  CanPress( Event e ) => true;     // Gate the press
    Tooltip? GetTooltip( Event e ) => null; // Optional UI hint
}
```

`Press` is the only required method; everything else has a default. `Press` **returns `bool`** — return `true` if the press was accepted (and you'll get a matching `Release` later), `false` to refuse silently.

## The lifecycle, in order

The player aims at the pressable, then either looks away or presses Use:

1. **`Hover`** — the player has just started looking at this.
2. **`Look`** — called every frame while they continue to look.
3. Then *one of two paths*:
   - **`Blur`** — they looked away, no press happened.
   - **`Press`** — they pressed Use (only fires if `CanPress` returned `true`). Continues into:
     1. **`Pressing`** — called every frame while the press is held. Return `false` to cancel mid-press.
     2. **`Release`** — Use was released, or `Pressing` returned `false`. Always called once after a successful `Press`.

Each callback receives an `IPressable.Event` — currently a record holding the `Component Source` (who triggered it; usually the `PlayerController`) and an optional `Ray` (the trace ray that found this pressable).

## Method semantics

| Method | Default | Return | Use it for |
|---|---|---|---|
| `Hover` | empty | — | Highlight the object, play a "look at me" sound. |
| `Look` | empty | — | **Per-frame** while the player aims at this. Update a holographic preview, refresh a text label. Cheap or it'll cost you. |
| `Blur` | empty | — | Stop the highlight. |
| `Press` | required | `bool` | The Use key was pressed. Return `true` to accept and *expect a Release*. Return `false` to ignore (e.g., not enough money). |
| `Pressing` | `true` | `bool` | **Per-frame** while held. Return `false` to cancel mid-press (e.g., the player ran out of fuel mid-cast). The engine will still call `Release` after a cancel. |
| `Release` | empty | — | The press is over. Always called after a successful `Press`, including after `Pressing` returned `false`. |
| `CanPress` | `true` | `bool` | Gate whether the player can press at all. Use this to grey out a tooltip ("Locked"). |
| `GetTooltip` | `null` | `Tooltip?` | The text + icon to show when the player aims at this. |

## The Tooltip

```csharp
public record struct Tooltip( string Title, string Icon, string Description,
                              bool Enabled = true, IPressable Pressable = default );
```

| Field | Purpose |
|---|---|
| `Title` | Short label, displayed prominently. |
| `Icon` | [Material icon](https://fonts.google.com/icons) name. |
| `Description` | Longer explanation, smaller text. |
| `Enabled` | When `false`, the tooltip renders greyed out — pair with `CanPress() => false`. |
| `Pressable` | Reserved; set automatically. Leave `default`. |

```csharp
public IPressable.Tooltip? GetTooltip( IPressable.Event e )
{
    if ( !HasFuel ) return new( "Locked", "lock", "Out of fuel.", Enabled: false );
    return new( "Drive", "drive_eta", "Get in" );
}
```

## Required setup on the GameObject

`IPressable` doesn't trace itself — the player controller does. For the player to *find* your pressable:

- The GameObject needs a `Collider` (or `MeshCollider` / `BoxCollider` / etc.) that intersects the player's interaction trace.
- The collider can be a trigger or a solid; the trace is configurable.
- The pressable component can be on the same GameObject as the collider, or anywhere in the hierarchy — most player controllers walk up to the GameObject root and then down to find an `IPressable`.

- **Always pair `Press` returning `true` with handling `Release`.** The interface contract says you'll receive Release; if you allocate anything in Press, free it in Release. **If `Press` returns `false`, you will *not* get a `Release`.**
- **Player dies mid-press.** The engine calls `Release` even if the player who started the press disconnects or dies. Don't assume `e.Source` is still valid.
- **`Pressing` returning `false` still triggers `Release`.** If you cancel a press from inside `Pressing` (return `false`), `Release` fires — useful for one-shot cleanup either way.
- **`Look` fires every frame.** Don't allocate or trace inside it. Cache state from `Hover`, update lightly in `Look`, clear in `Blur`.
- **Tooltips re-evaluate constantly.** `GetTooltip` is called frequently while hovering. It should be cheap — return cached strings when possible.

## See also

- [Component Interfaces overview](index.md)
- [Player Controller component](../reference/player-controller.md) — the typical caller.
- [Chair component](../reference/chair.md) — a working example of `IPressable` for sittable seats.
