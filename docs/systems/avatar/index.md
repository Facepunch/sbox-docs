---
title: "Avatar & Clothing"
icon: "♿"
created: 2024-04-12
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Game/Avatar/ClothingContainer.cs
  - engine/Sandbox.Engine/Game/Avatar/ClothingContainer.Dressing.cs
---

# Avatar & Clothing

The Avatar system allows you to dress a `SkinnedModelRenderer` in clothing, using either a specific local player's setup or programmatically assigned outfits.

## Quick Working Example

```csharp
using Sandbox;
using System.Threading;
using System.Threading.Tasks;

public sealed class PlayerDresser : Component
{
    [RequireComponent] public SkinnedModelRenderer Body { get; set; }

    protected override async Task OnStart()
    {
        // 1. Get the local user's clothing selection from their profile
        var clothing = ClothingContainer.CreateFromLocalUser();

        // 2. Apply it to the SkinnedModelRenderer component
        // Using ApplyAsync ensures any remote assets are downloaded first
        await clothing.ApplyAsync( Body, CancellationToken.None );
    }
}
```

### Dressing from a Network Connection

If you are making a multiplayer game, you typically want to dress characters based on each connected player's avatar. You can get a specific player's clothing when they join.

```csharp
public void DressPlayer( Connection connection, SkinnedModelRenderer body )
{
    // Load the clothing from this specific connection's profile
    var clothing = ClothingContainer.CreateFromConnection( connection );

    // Apply the outfit
    clothing.Apply( body );
}
```

### Custom Programmatic Outfits

Sometimes you don't want players to use their own avatars—maybe you want everyone to wear a specific team uniform, or you are generating NPCs. You can manually build a `ClothingContainer` from scratch.

```csharp
public sealed class UniformDresser : Component
{
    [RequireComponent] public SkinnedModelRenderer Body { get; set; }

    [Property] public Clothing Shirt { get; set; }
    [Property] public Clothing Pants { get; set; }

    protected override void OnStart()
    {
        var container = new ClothingContainer();

        // Add our specific clothing items
        if ( Shirt != null ) container.Toggle( Shirt );
        if ( Pants != null ) container.Toggle( Pants );

        // Apply it to the character
        container.Apply( Body );
    }
}
```

### Downloading Missing Outfits

If an outfit includes an item from an external package (like a community workshop item), calling `Apply()` immediately might skip it if the model hasn't downloaded yet. To ensure all missing assets are fetched before dressing, you should await `ApplyAsync()`.

```csharp
public async Task DressPlayerFully( SkinnedModelRenderer body )
{
    var clothing = ClothingContainer.CreateFromLocalUser();

    // Wait until any missing models or packages have downloaded
    await clothing.ApplyAsync( body );
}
```

## Performance

*   **Apply is Expensive:** Calling `Apply()` or `ApplyAsync()` involves generating new GameObjects for the clothes and mapping their bones to the parent `SkinnedModelRenderer`. Do not call this in an `OnUpdate` loop. Dress a character once when they spawn, or only when their outfit actually changes.
*   **Draw Calls:** While the clothing system is optimized, every piece of clothing adds draw calls. For large crowds of distant NPCs, consider baking standard models or manually combining meshes.

*   **Bodygroups:** Clothing items can specify that they hide certain body parts (e.g., a jacket hiding the underlying torso). If you bypass `ClothingContainer` and just child a clothing model to the player, you will see clipping because the underlying bodygroups weren't updated. Always use `ClothingContainer.Apply()`.

## Troubleshooting

:::danger "The character is naked / Missing their body!"
The base `SkinnedModelRenderer` must have a valid model assigned (usually `models/citizen/citizen.vmdl`) for the clothing system to attach to properly. Also, if you use `Apply` instead of `ApplyAsync`, network-streamed clothing from custom packages might be missing until downloaded.
:::

:::warning "Weird overlapping clothes"
`ClothingContainer.Apply()` automatically handles hiding body parts (like hiding the chest if wearing a jacket) and generating the right sub-objects. Do not manually child clothing models to the player yourself unless you are writing a completely custom system.
:::

## Sample Projects

*   [**Multiplayer Testbed**](../../build-games/samples-templates/index.md) - Contains working examples of spawning players and applying their connection's clothing container.
