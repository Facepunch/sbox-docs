---
title: "Command Lists"
icon: "📋"
created: 2024-12-08
updated: 2025-08-11
---

# Command Lists

Command Lists are deferred commands to execute on the renderer on certain rendering stage, this replaces the previous `RenderHook` system

```csharp
public enum Stage
{
	AfterDepthPrepass = 1000,
	AfterOpaque = 2000,
	AfterSkybox = 3000,
	AfterTransparent = 4000,
	AfterViewmodel = 5000,
	BeforePostProcess = 6000,
	AfterPostProcess = 7000,
	AfterUI = 8000,
}
```

You can attach a Command List to a Camera, each command that you add to the command list will be executed in sequence, and will execute for any camera that depends on it ( Editor Viewport will inherit from main camera, etc )

```csharp
	protected override void OnEnabled()
	{
		commands = new Rendering.CommandList( "AmbientOcclusion" );
  
        // Build your commands here eg
        commands.Set("Foo", 1.0f );
        
		Camera.AddCommandList( commands, Rendering.Stage.AfterDepthPrepass );
	}
```

And to remove it

```csharp
protected override void OnDisabled()
{
    Camera.RemoveCommandList( commands );
    commands = null;
}
```

See [Attributes and Variables](attributes-and-variables.md) for an example of using a Command List
