---
title: "Create a UI Health Bar"
icon: "❤️"
sources:
  - engine/Sandbox.Engine/Scene/Components/UI/PanelComponent.cs
updated: 2026-04-25
created: 2026-04-27
---

# Create a UI Health Bar

> **New to UI?** This is the **recipe** for one specific task: building a UI health bar. Full UI documentation at **[UI](../systems/ui/index.md)**.

How to create a simple, responsive health bar that updates when a player takes damage using Razor Panels.

## Quick Working Example

Create a new file called `HealthBar.razor` and add it as a component to a `GameObject` that also has a `ScreenPanel`.

```csharp
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="health-bar-container">
        <div class="health-bar-fill" style="width: @(HealthPercentage)%"></div>
        <label>@CurrentHealth / @MaxHealth</label>
    </div>
</root>

@code
{
    [Property] public int CurrentHealth { get; set; } = 75;
    [Property] public int MaxHealth { get; set; } = 100;

    // Calculate percentage for the CSS width
    private float HealthPercentage => (float)CurrentHealth / MaxHealth * 100f;

    // Tell the UI to redraw if Health changes
    protected override int BuildHash() => System.HashCode.Combine( CurrentHealth, MaxHealth );
}

<style>
    .health-bar-container {
        position: absolute;
        bottom: 50px;
        left: 50px;
        width: 300px;
        height: 30px;
        background-color: rgba( 0, 0, 0, 0.5 );
        border-radius: 4px;
        overflow: hidden;
        align-items: center;
        justify-content: center;
    }

    .health-bar-fill {
        position: absolute;
        left: 0;
        top: 0;
        bottom: 0;
        background-color: red;
        transition: width 0.2s ease;
    }
    
    label {
        color: white;
        font-family: Poppins;
        font-weight: bold;
        z-index: 1;
    }
</style>
```

### Reading Health from Another Component

In a real game, your UI won't store the health variable directly. It will read it from a `Player` or `Health` component on another GameObject.

```csharp
@code
{
    // Drag your player GameObject into this property in the Editor
    [Property] public GameObject PlayerObject { get; set; }

    // Optional: caching the health component
    private MyHealthComponent playerHealth;

    protected override void OnStart()
    {
        if ( PlayerObject != null )
        {
            playerHealth = PlayerObject.Components.Get<MyHealthComponent>();
        }
    }

    private float HealthPercentage 
    {
        get
        {
            if ( playerHealth == null ) return 0;
            return (float)playerHealth.CurrentHealth / playerHealth.MaxHealth * 100f;
        }
    }

    protected override int BuildHash() 
    {
        // Rebuild whenever the player's actual health changes
        return playerHealth != null ? playerHealth.CurrentHealth : 0;
    }
}
```

## Troubleshooting

:::danger "The health bar isn't updating!"
Ensure you have overridden `BuildHash()`. Razor UI does not redraw every frame by default for performance reasons. It only redraws when the value returned by `BuildHash()` changes. 
:::

:::warning "The fill goes outside the container!"
Make sure your `.health-bar-container` has `overflow: hidden;` in its CSS. This ensures that even if the fill somehow exceeds 100% (e.g. from overhealing), it won't draw outside the bounds of the background.
:::

## Related Pages
- [Razor Panels](../systems/ui/razor-panels/index.md)
- [Screen Panel](../systems/ui/screenpanel.md)
