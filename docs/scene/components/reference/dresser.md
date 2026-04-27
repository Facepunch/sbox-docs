---
title: "Dresser"
icon: "👕"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/Dresser.cs
updated: 2026-04-25
created: 2026-04-27
---

# Dresser

> **Looking up a component?** Component Reference index is at **[Built-in Components](index.md)**. This page documents the `Dresser` component.

`Dresser` applies clothing items to a `SkinnedModelRenderer` representing a citizen or human body. It's the standard way to dress a character in s&box — either with manually-picked items, the local user's avatar, or the avatar of the network owner.

## Quick Working Example

```csharp
var citizen = Scene.CreateObject();
var renderer = citizen.AddComponent<SkinnedModelRenderer>();
renderer.Model = Model.Load( "models/citizen/citizen.vmdl" );

var dresser = citizen.AddComponent<Dresser>();
dresser.BodyTarget = renderer;
dresser.Source = Dresser.ClothingSource.OwnerConnection;
```

When the scene starts, `Dresser` automatically calls `Apply()` and the citizen takes on the player's avatar.

## Properties

| Property | Description |
|:---|:---|
| `Source` | `Manual`, `LocalUser`, or `OwnerConnection`. Determines where the clothing comes from. |
| `RemoveUnownedItems` | When `OwnerConnection`, strip clothing items the owner doesn't own in their Steam Inventory. Default `true`. Disable only if your game does its own ownership checks. |
| `BodyTarget` | The `SkinnedModelRenderer` to dress. Auto-found from descendants in `OnValidate` if unset. |
| `ApplyHeightScale` | If true, applies the height attribute as a body scale. Default `true`. |
| `ManualHeight` / `ManualTint` / `ManualAge` | `0..1` sliders used when `Source == Manual`. Networked and re-applied via `ApplyAttributes` on proxies. |
| `Clothing` | List of `ClothingContainer.ClothingEntry` for manual outfits. |
| `WorkshopItems` | List of workshop package idents to install before applying clothing. |

## Common Patterns

### Network-owned dressing

For each player's character, set `Source = OwnerConnection`. Each player's owning client computes and applies its own outfit; proxies receive the height/age/tint values as networked attributes and don't re-fetch workshop content.

### Editor preview

While editing in the scene editor, change `ManualHeight`, `ManualTint`, `ManualAge`, or the `Clothing` list — `OnValidate` triggers `Apply` and the preview updates live.

### Random outfit button

```csharp
[Button, ShowIf( nameof( Source ), ClothingSource.Manual )]
public void Randomize() // built into Dresser
```

The component already exposes a `Randomize` button in Manual mode that picks a random outfit via `AvatarRandomizer.GetRandom()` and randomises the height/age/tint sliders.

:::warning "Apply is async"
`Apply()` returns a `ValueTask`. When workshop items are involved it can take seconds. If you destroy the `GameObject` during dressing, the internal `CancellationTokenSource` cancels the in-flight work cleanly — but if you `await Apply()` yourself, handle cancellation.
:::

:::note "MergeDescendants is destructive"
After applying, `BodyTarget.MergeDescendants()` collapses spawned clothing renderers into a single mesh hierarchy. Don't expect to find individual clothing item `GameObject`s as children afterwards.
:::

## Related Pages

- [Skinned Model Renderer](skinnedmodelrenderer.md)
- [Player Controller](player-controller.md)
