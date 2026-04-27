---
title: "Standalone Builds"
sources:
  - engine/Sandbox.Engine/Platform/Standalone/
created: 2026-04-27
updated: 2026-04-27
---

# Standalone Builds

When you export your game to distribute outside of the s&box ecosystem, it runs as a **Standalone** build. The `Standalone` API provides access to the build manifest, build dates, and handles initializing the application environment without the s&box developer editor.

## Quick Start

```csharp
using Sandbox;

public sealed class StandaloneCheckComponent : Component
{
    protected override void OnStart()
    {
        if ( Application.IsStandalone )
        {
            Log.Info( $"Running standalone build from {Standalone.BuildDate}" );
        }
    }
}
```

## Common Patterns

### Disabling Debug Features
Often you will have developer-only menus or debugging tools that should not be visible to players in a published build. You can use the `Application.IsStandalone` flag to easily hide these features.

```csharp
public sealed class DebugMenu : Component
{
    protected override void OnStart()
    {
        // Destroy the debug menu if we're in a standalone build
        if ( Application.IsStandalone )
        {
            GameObject.Destroy();
        }
    }
}
```

## Troubleshooting

:::warning Standalone Missing Assets
If your Standalone build is missing assets or failing to load certain files, ensure that they are properly referenced and included in your build configuration. The Standalone environment does not have access to the full s&box editor asset library.
:::

## Related Pages
* [Platform Integration](./index.md)
* [Exporting Standalone](../../publishing/exporting-standalone.md)
