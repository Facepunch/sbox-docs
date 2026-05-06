---
title: "IScenePhysicsEvents"
icon: "🚪"
created: 2024-10-30
updated: 2024-10-30
---

# IScenePhysicsEvents

This interface allows you to wrap logic around the physics step. This can be useful when you have an object, or a system, that works closely with the physics system.


```csharp
public interface IScenePhysicsEvents : ISceneEvent<IScenePhysicsEvents>
{
	/// <summary>
	/// Called before the physics step is run. This is called pretty much
	/// right after FixedUpdate.
	/// </summary>
	void PrePhysicsStep() { }

	/// <summary>
	/// Called after the physics step is run
	/// </summary>
	void PostPhysicsStep() { }
}
```
