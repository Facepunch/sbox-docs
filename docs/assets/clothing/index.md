---
title: "Clothing"
icon: "🩲"
created: 2025-01-14
updated: 2025-06-15
sources:
  - engine/Sandbox.Engine/Game/Avatar/Clothing.cs
  - engine/Sandbox.Engine/Game/Avatar/ClothingContainer.cs
  - engine/Sandbox.Engine/Game/Avatar/ClothingContainer.Dressing.cs
  - engine/Sandbox.Engine/Scene/Components/Game/Dresser.cs
---

# Clothing

The Clothing system allows you to define, load, and equip custom clothing items on the base player models (Citizen and Humans).

## Quick Working Example

```csharp
using Sandbox;

public sealed class PlayerOutfitSetup : Component
{
	[Property] public SkinnedModelRenderer BodyRenderer { get; set; }
	[Property] public Clothing OutfitItem { get; set; }

	protected override void OnStart()
	{
		// 1. Create an empty container
		var clothingContainer = new ClothingContainer();

		// 2. Add our custom clothing item
		if ( OutfitItem != null )
		{
			clothingContainer.Add( OutfitItem );
		}

		// 3. Apply the clothes to our character's SkinnedModelRenderer
		clothingContainer.Apply( BodyRenderer );
	}
}
```

1. **The Dresser Component:**
   The easiest way to apply clothing to a character without writing code is to use the built-in **`Dresser`** component in the Editor.
   - Add a `Dresser` component to your GameObject.
   - Assign the character's `SkinnedModelRenderer` to the component's body target property.
   - Change the source property to **LocalUser** to automatically dress the character in the outfit the local player has configured in their main menu settings, or **Manual** to pick specific `.clothing` files yourself.

2. **Loading and Toggling Clothing via Code:**
   You can fetch clothing resources from your project and toggle them on a character dynamically during gameplay (e.g., finding a hat on the ground).
   ```csharp
   public void ToggleHat( SkinnedModelRenderer renderer )
   {
   	var container = new ClothingContainer();
   	
   	// Load the clothing resource from disk
   	if ( ResourceLibrary.TryGet<Clothing>( "models/hats/cool_hat.clothing", out var hat ) )
   	{
   		container.Toggle( hat ); // Toggles the hat on or off
   		container.Apply( renderer ); // Re-apply to update visuals
   	}
   }
   ```

3. **Cloud Avatars:**
   Instead of defining local clothing, you can fetch a specific player's outfit directly from the asset.party backend using their SteamID.
   ```csharp
   var container = new ClothingContainer();
   container.LoadFromClient( Connection.Local ); // Load local player's avatar
   container.Apply( BodyRenderer );
   ```

## Navigation Hub

* [**First Time Setup**](first-time-setup.md) - Get the base character files to start modeling clothes.
* [**Guidelines & Quality Bar**](guidelines-quality-bar.md) - Learn the requirements and limits for clothing assets.
* [**Layering Clothing**](layering-clothing.md) - Understand how different clothing layers interact without clipping.
* [**Making a Hat**](making-a-hat.md) - A simple guide to creating your first clothing item.
* [**Morphing to Humans**](morphing-to-humans.md) - How to adapt Citizen clothing to fit the Human models.
* [**Publishing Clothing**](publishing-clothing.md) - Setup your `.clothing` file and publish it to the s&box backend.

## Troubleshooting

:::warning Models Not Following the Character
If your clothing item appears on the ground or doesn't animate with the character, ensure it was exported with the correct skeletal rig (the standard `citizen` or `human` skeleton). The `ClothingContainer` relies on matching bone names exactly to merge the clothing mesh.
:::

:::warning Missing References / Null Clothing
If `ResourceLibrary.TryGet<Clothing>` returns false, ensure your `.clothing` file is compiled and included in your project, and that the path you provided is exactly correct (e.g., `"models/my_shirt.clothing"`). Remember paths are relative to the addon root.
:::

:::warning Clothing Clipping Through Body
Ensure your clothing asset has proper **Hidebody** data configured in its `.clothing` file. This tells the base Citizen model which parts of its body mesh to hide (e.g., hiding the legs when wearing pants) to prevent the skin from clipping through the clothes during animations.
:::

