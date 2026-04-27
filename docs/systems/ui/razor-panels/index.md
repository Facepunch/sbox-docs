---
title: "Razor Panels"
icon: "⬜"
created: 2024-09-24
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Scene/Components/UI/PanelComponent.cs
  - engine/Sandbox.Engine/Systems/UI/Panel/Panel.cs
---

# Razor Panels

> **New to UI?** Reference for Razor panel syntax; UI overview is at **[UI](../index.md)**.

Razor Panels let you create and style UI elements using familiar HTML/CSS syntax while seamlessly injecting C# code for logic and data binding.

## Quick Working Example

Create a new "Razor Panel Component" in the Editor and attach it to a GameObject with a `ScreenPanel`.

```csharp
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
	<div class="health-bar">
		<label>HP: @Health</label>
		<button onclick="@TakeDamage">Ouch!</button>
	</div>
</root>

@code
{
	[Property] public int Health { get; set; } = 100;

	void TakeDamage()
	{
		Health -= 10;
	}

	/// <summary>
	/// If the value returned from BuildHash changes, the UI will be rebuilt.
	/// </summary>
	protected override int BuildHash() => System.HashCode.Combine( Health );
}
```

### Injecting C# Logic into Layout

You can use standard C# control flow (`foreach`, `if`) directly inside the layout block by prefixing them with `@`.

```markup
<root>
  @foreach(var player in Scene.GetAllComponents<PlayerController>())
  {
    <div class="player-card">
      <label>@player.GameObject.Name</label>
      
      @if(player.Health <= 0)
      {
        <img src="ui/skull.png" />
      }
    </div>
  }
</root>
```

### Nesting Panels

To keep your UI organized, you should break it into smaller panels. If you create a `PlayerCard.razor` (which inherits from `Panel`, not `PanelComponent`), you can use it inside another Razor file just like an HTML tag.

```csharp
<root>
  <!-- Passing data into the child panel's properties -->
  <PlayerCard Name="Alice" Health=@(100) />
  <PlayerCard Name="Bob" Health=@(50) />
</root>
```

### Getting a Reference to a Child Panel

If you need to call methods on a child panel from C#, use the `@ref` attribute.

```csharp
<root>
  <MyChildPanel @ref="PanelReference" />
</root>

@code
{
  MyChildPanel PanelReference { get; set; }
  
  protected override void OnStart()
  {
    PanelReference.SetData( "Setup complete!" );
  }
}
```

### Two-Way Data Binding

If you have a UI control (like a text input or slider) and want a C# variable to automatically update when the user changes it (and vice-versa), use `:bind`.

```csharp
<SliderEntry min="0" max="100" step="1" Value:bind=@MyVolume></SliderEntry>

@code
{
	public float MyVolume { get; set; } = 50f;
	
	protected override int BuildHash() => System.HashCode.Combine( MyVolume );
}
```

## Configuration

When creating a `.razor` file, you choose what it inherits from at the top of the file (e.g., `@inherits PanelComponent`).

| Type | Description | Key Differences |
|---|---|---|
| **PanelComponent** | The root of a UI hierarchy. Attaches to a GameObject. | Has `OnUpdate`, `OnStart`. Modifies its panel via the `Panel` property (e.g., `Panel.Style.Left`). |
| **Panel** | A standard UI element nested inside a PanelComponent. | Has `Tick`, `OnAfterTreeRender`. Modifies its own style directly (e.g., `Style.Left`). Cannot be added to a GameObject. |

## Troubleshooting

:::danger "My UI isn't updating when variables change!"
If your variables change but the screen doesn't update, you likely forgot to include those variables in the `BuildHash()` method. Razor only redraws the UI if the value returned by `BuildHash()` changes.
:::

:::danger "I can't add my Panel to a GameObject!"
If you get an error trying to attach a UI script to a GameObject in the Inspector, ensure the `.razor` file has `@inherits PanelComponent` at the top, not `Panel`.
:::

:::warning "I'm getting errors setting Style.Width in my PanelComponent!"
Because `PanelComponent` is a *wrapper* around a Panel (not a Panel itself), you must access the inner panel first: `Panel.Style.Width = Length.Pixels(100);`.
:::

## Related Pages
* [UI Overview](../index.md)
* [Razor Components](razor-components.md)
