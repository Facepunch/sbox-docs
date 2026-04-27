---
title: Scene System Architecture
icon: "🧠"
sources:
  - engine/Sandbox.Engine/Scene/
updated: 2026-04-25
created: 2026-04-27
---

# Scene System Architecture

The Scene System is the foundational layer of s&box game logic. It manages the hierarchy of `GameObjects`, the execution of `Component` lifecycles, and acts as the managed bridge to the native Source 2 rendering and physics pipelines.

## Quick Start Example

As an engine contributor, you may need to interact with the underlying Scene tick loop or system registration:

```csharp
using Sandbox;

// Conceptual: Engine-level interaction with a Scene
internal class SceneDebugger
{
    public void DumpSceneGraph( Scene scene )
    {
        Log.Info( $"Dumping {scene.GetAllObjects(true).Count()} objects in {scene.Name}" );
        
        foreach ( var obj in scene.GetAllObjects( true ) )
        {
            if ( !obj.Enabled ) continue;
            
            // Accessing internal flags or iterating components
            Log.Info( $"- {obj.Name} has {obj.Components.Count} components." );
        }
    }
}
```

## Common Patterns

1. **Component Lifecycle:** The engine strictly enforces `OnAwake` -> `OnStart` -> `OnUpdate`/`OnFixedUpdate` -> `OnDestroy`. This predictable flow is managed centrally by the Scene.
2. **Transform Hierarchy:** When a parent GameObject moves, the engine calculates the dirty local-to-world matrices and propogates the changes down to all children before the next render pass.
3. **Prefab Instantiation:** The `PrefabSystem` reads serialized JSON configurations and rapidly constructs GameObject hierarchies in memory without needing to compile code.

## Troubleshooting

:::warning
- **"Transform cannot be assigned to a child of itself":** The engine prevents circular dependencies in the Scene Graph. Attempting to parent an object to its own child will throw an exception to prevent infinite recursive loops during matrix calculation.
- **Component Execution Order:** The engine does not guarantee the order in which `OnUpdate` is called across different GameObjects unless explicitly configured via `[Feature]` dependencies. Do not write code that assumes Object A will tick before Object B.
:::

## Related Pages
- [Sandbox.Engine](../../engine-internals/sandbox-engine/index.md)
