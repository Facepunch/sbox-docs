---
title: "Component Methods"
icon: "👷"
created: 2023-12-28
updated: 2026-06-08
---

# Component Methods

When creating a component there are a number of methods you can override and implement.


:::info
Note that for the component be enabled, its GameObject and all of their ancestors need to be enabled too. The GameObject will be considered disabled if one of its ancestor GameObjects is not enabled.

:::

# OnLoad (async)

This is called after deserialization and is meant for a place for the component to "load". When loading a scene, the loading screen will stay open and the game won't start until all components `OnLoad` tasks are complete. 

Unlike other methods (except `OnDestroy`), this one will execute even if the component is inactive. You may need to check `Component.Active` if you don't want the OnLoad to execute on inactive components.

If your component is doing something special, such as generating a procedural level, you can override this on your component to do this in the loadscreen.

```csharp
protected override async Task OnLoad()
{
	LoadingScreen.Title = "Loading Something..";
	await Task.DelayRealtimeSeconds( 1.0f );
}
```


:::info
Internally this is where the Map component downloads and loads the map.

:::

# OnAwake

Called once when the component is created, but only if our parent GameObject is enabled. This is called after deserialization and loading.

# OnStart

Called when the component is enabled for the first time. Should always get called before the first `OnFixedUpdate`.

# OnEnabled

Called when the component is enabled. 

# OnUpdate

Called every frame.

# OnPreRender

:::info
This method is not called on dedicated servers.

:::

Called every frame, right before rendering is about to take place.

This is called after animation bones have been calculated, so it usually a good place to do things that count on that.

# OnFixedUpdate

Called every fixed timestep.

In general, it's wise to use a fixed update for things like player movement (the built in Character Controller does this). This reduces the amount of traces a client is doing every frame, and if your client is too performant, the move deltas per frame can be so small that they create problems. 

# OnDisabled

Called when the component is disabled.

# OnDestroy

Called when the component is destroyed. This method will be called even if the component was never enabled.
