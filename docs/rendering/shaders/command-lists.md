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
	EarlyUI = 5500,
	BeforePostProcess = 6000,
	Tonemapping = 6500,
	AfterPostProcess = 7000,
	UI = 7500,
	AfterUI = 8000,
}
```

You can attach a Command List to a Camera, each command that you add to the command list will be executed in sequence, and will execute for any camera that depends on it ( Editor Viewport will inherit from main camera, etc )

```csharp
	protected override void OnEnabled()
	{
		commands = new Rendering.CommandList( "AmbientOcclusion" );
  
        // Build your commands here eg
        commands.Attributes.Set("Foo", 1.0f);
        
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

See [Attributes and Variables](/rendering/shaders/attributes-and-variables.md) for an example of using a Command List

# GPU Profiling Scopes

Command lists on their own already appear in GPU profiling (tools like RenderDoc, or GPU profiler in `overlay_gpu`), but you can also create your own custom scopes inside that command list, which will calculate timings for everything happening in that scope. This is how you can declare a new profiling scope and add code within it:

```csharp
static readonly ProfilingSampler MyProfilingScope = new( "My Profiling Scope" );

// later in the actual command list code:
using ( _commandList.ProfileScope( MyProfilingScope ) )
{
    // do something here...
}
```

Profiling also supports nested scopes.

```csharp
// This scope will appear in profiling like this:
//
// PARENT SCOPE
//	- NESTED SCOPE A
//	- NESTED SCOPE B
using ( _commandList.ProfileScope( ParentScope ) ) 
{
	using ( _commandList.ProfileScope( NestedScopeA ) )
	{
		// something here...
	}

	using ( _commandList.ProfileScope( NestedScopeB ) ) 
	{
		// something here....
	}
}
```