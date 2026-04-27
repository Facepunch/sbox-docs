---
title: Razor UI Compiler Architecture
icon: "💻"
sources:
  - engine/Sandbox.Razor
  - engine/Microsoft.AspNetCore.Components
created: 2026-04-27
updated: 2026-04-27
---

# Razor UI Compiler Architecture

Integrates Microsoft's Razor templating engine to allow developers to write user interfaces using HTML/CSS interspersed with C#. This page explains the internal compilation pipeline and how it transforms markup into a runtime C# object tree.

## Quick Start Example

Here is a simple component written in Razor:

```html
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root>
    <div class="health-bar" style="width: @(PlayerHealth)%"></div>
    <label>Health: @PlayerHealth</label>
</root>

@code
{
    public float PlayerHealth { get; set; } = 100f;

    // BuildHash tells the compiler when to re-render the UI
    protected override int BuildHash() => System.HashCode.Combine( PlayerHealth );
}
```

## Internal Workflow

1.  **Parse:** The `.razor` file is read by the `Sandbox.Razor` system.
2.  **Generate:** C# code is generated, inheriting from `PanelComponent` (or whatever is specified).
3.  **Compile:** The generated C# is compiled alongside your regular `.cs` scripts into your game assembly.
4.  **Render:** At runtime, when the panel is created, it executes the generated `BuildRenderTree` method, building the visual UI.

## Performance

-   **BuildHash is the Gateway:** The `BuildHash()` method is critical. The Razor system checks this hash every frame. It only re-executes the expensive `BuildRenderTree` function (which reconstructs the UI tree) if the hash has changed. **Do not** return `Time.Now` or a random number, or you will force a UI rebuild every frame, destroying performance.
-   **Static Elements:** If a panel's contents never change, you can return a constant hash (e.g., `1`), and it will render exactly once.

## Troubleshooting

:::warning Common Gotchas
- **Compile Errors pointing to invisible code:**
  Sometimes you will get a C# compile error that points to a line number that doesn't exist in your `.razor` file. This is because the error is in the *generated* C# code. Usually, this means you missed a closing brace `}` in an `@code` block, or misspelled a variable name in an `@` expression.
- **UI not updating:**
  If your variable changes but the screen doesn't, check your `BuildHash()`. Ensure the variable you modified is actually included in the `HashCode.Combine()` call.
:::

## Related Pages
- [UI Styling Panels](../styling-panels/index.md)
- [Code Hotloading](../../../scripting-code-workflows/code-hotloading.md)
