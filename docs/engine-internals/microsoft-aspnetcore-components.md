---
title: "Microsoft.AspNetCore.Components"
icon: "💻"
sources:
  - engine/Microsoft.AspNetCore.Components/
created: 2026-04-27
updated: 2026-04-27
---

# Microsoft.AspNetCore.Components

The `engine/Microsoft.AspNetCore.Components/` directory contains stubs and simplified implementations of standard ASP.NET Core Razor interfaces, adapted specifically for the engine's custom UI compiler pipeline.

## Quick Start Example

As an engine developer, you will rarely modify this directory directly. These files exist primarily to trick standard IDEs (like Visual Studio and Rider) into providing full IntelliSense and syntax highlighting for `.razor` files inside s&box without needing the full, heavyweight ASP.NET Core framework.

```csharp
using Microsoft.AspNetCore.Components.Rendering;

// Conceptual: A simplified RenderTreeBuilder stub provided by this directory.
// The actual s&box Razor compiler will map these method calls to its own internal
// DOM builder during compilation.
public abstract partial class RenderTreeBuilder
{
    public abstract void OpenElement( int sequence, string elementName );
    public abstract void CloseElement();
}
```

## Troubleshooting

:::warning Conflicting Namespaces
If a game developer attempts to manually import `Microsoft.AspNetCore.Components` from NuGet into their s&box project, they will immediately encounter massive compilation errors and ambiguity conflicts with this built-in engine assembly. Developers must never import standard ASP.NET Core web libraries into an s&box project.
:::

## Related Pages
- [Sandbox.Razor](sandbox-razor.md)
- [Razor UI Compiler Architecture](../systems/ui/razor/architecture.md)
