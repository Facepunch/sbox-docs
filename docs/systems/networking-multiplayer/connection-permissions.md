---
title: "Connection Permissions"
icon: "🔑"
created: 2024-03-21
updated: 2024-04-11
sources:
  - engine/Sandbox.Engine/Systems/Networking/System/Channel/Connection.cs
---

# Connection Permissions

The host can configure what specific connected players (`Connection`) are allowed to do with networked objects, such as whether they can spawn, refresh, or destroy them.

## Quick Working Example

The ideal place to set these permissions is in the `OnActive` method of a component implementing `Component.INetworkListener`.

```csharp
public sealed class PermissionManager : Component, Component.INetworkListener
{
	public void OnActive( Connection connection )
	{
		// Grant or restrict permissions for the newly connected player
		if ( !connection.IsHost )
		{
			// Example: Prevent clients from spawning their own objects
			connection.CanSpawnObjects = false;
			
			// Example: Allow clients to refresh and destroy objects they own
			connection.CanRefreshObjects = true;
			connection.CanDestroyObjects = true;
		}
	}
}
```

## Configuration Options

These properties must be set by the **host**. If a client tries to change them, the changes will be ignored.

| Property | Default | Description |
|---|---|---|
| `CanSpawnObjects` | `true` | Allows this connection to create new networked objects (via `GameObject.NetworkSpawn()`). |
| `CanRefreshObjects` | `true` | Allows this connection to send network refresh updates for objects they **own**. |
| `CanDestroyObjects` | `true` | Allows this connection to destroy networked objects they **own**. |

## Troubleshooting

:::warning Unauthorized Action
If a client attempts to destroy or refresh an object they own, but their `Connection` does not have the corresponding permission (`CanDestroyObjects` or `CanRefreshObjects` is false), the request is ignored and a warning is logged on the server.
:::
