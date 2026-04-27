---
title: "Network Visibility"
icon: "👁️"
created: 2025-11-14
updated: 2026-04-13
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkVisible.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.Network.cs
---

# Network Visibility

Network visibility allows you to control which connected players receive network updates (like transform movement and synced variables) for specific objects in your scene.

## Quick Working Example

To control visibility, add a component that implements the `Component.INetworkVisible` interface to the root of your networked `GameObject`, and turn off **Always Transmit**.

```csharp
using Sandbox;

public sealed class TeamVisibility : Component, Component.INetworkVisible
{
	[Property] public int TeamId { get; set; } = 1;

	protected override void OnStart()
	{
		// 1. Tell the network system we want to manually control visibility
		GameObject.Network.AlwaysTransmit = false;
	}

	// 2. This method is called by the engine for every connected client
	bool Component.INetworkVisible.IsVisibleToConnection( Connection connection, in BBox worldBounds )
	{
		// In this example, assume we have a way to check the connection's team
		int connectionTeam = connection.GetUserData("team_id") as int? ?? 0;

		// Only transmit updates to players on the same team
		return connectionTeam == TeamId;
	}
}
```

### Always Transmit

You can easily toggle whether an object transmits to everyone using the `GameObject.Network.AlwaysTransmit` property. By default, this is `true`.

```csharp
// Stop forcing updates to everyone. We want to use custom visibility or distance culling!
GameObject.Network.AlwaysTransmit = false;
```

### Distance-Based Culling

One of the most common reasons to cull objects is because they are too far away for the player to see. You can use the `worldBounds` parameter provided in `IsVisibleToConnection` to check the distance.

```csharp
public sealed class DistanceCulling : Component, Component.INetworkVisible
{
	[Property] public float MaxVisibleDistance { get; set; } = 4000f;

	protected override void OnStart()
	{
		GameObject.Network.AlwaysTransmit = false;
	}

	bool Component.INetworkVisible.IsVisibleToConnection( Connection connection, in BBox worldBounds )
	{
		// Find the player object associated with this connection
		var playerCamera = GetPlayerCameraForConnection( connection );
		if ( playerCamera == null ) return false;

		// Calculate the distance
		float distance = playerCamera.WorldPosition.Distance( worldBounds.Center );
		
		// Cull the object if it's too far away
		return distance <= MaxVisibleDistance;
	}
	
	private GameObject GetPlayerCameraForConnection( Connection connection )
	{
		// ... your logic to find the connection's active camera/player ...
		return null;
	}
}
```

## Troubleshooting

:::warning My object disappears immediately!
If you turn `AlwaysTransmit` off but forget to add a component that implements `INetworkVisible` (or if your logic returns `false`), the object will be culled and instantly disabled for clients!
:::

:::danger IsVisibleToConnection isn't being called!
`IsVisibleToConnection` is only executed on the **Host** (the server). The clients don't get to decide what they see—the server makes that decision for them. Furthermore, it won't be called if `AlwaysTransmit` is set to `true`.
:::

## Related Pages
- [Networked Objects](networked-objects.md)
- [Sync Properties](sync-properties.md)
