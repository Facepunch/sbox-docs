---
title: "Custom Snapshot Data"
icon: "💿"
created: 2024-08-17
updated: 2026-04-14
sources:
  - engine/Sandbox.Engine/Scene/Components/Markers/INetworkSnapshot.cs
---

# Custom Snapshot Data

The `Component.INetworkSnapshot` interface allows your components to write and read large blocks of custom data during the initial network snapshot when a client joins a server.

## Quick Working Example

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class VoxelWorldData : Component, Component.INetworkSnapshot
{
	private byte[] _voxelData;

	// 1. Write the data into the network stream for new clients
	void Component.INetworkSnapshot.WriteSnapshot( ref ByteStream writer )
	{
		if ( _voxelData == null )
		{
			writer.Write( 0 );
			return;
		}

		writer.Write( _voxelData.Length );
		writer.WriteArray( _voxelData );
	}

	// 2. Read the data from the network stream when joining
	void Component.INetworkSnapshot.ReadSnapshot( ref ByteStream reader )
	{
		var length = reader.Read<int>();
		if ( length > 0 )
		{
			_voxelData = reader.ReadArray<byte>( length ).ToArray();
		}
	}
}
```

### Waiting for Data to Load

Sometimes, the data you receive in the snapshot takes time to process (like generating a mesh from voxel bytes). You don't want the client to finish joining and start playing while the world is still building. You can delay the load by returning a `Task` from the component's `OnLoad()` or `OnStart()` methods, which will hold the loading screen until it completes.

```csharp
public sealed class WorldGenerator : Component, Component.INetworkSnapshot
{
	private byte[] _worldData;

	void Component.INetworkSnapshot.WriteSnapshot( ref ByteStream writer )
	{
		if ( _worldData == null )
		{
			writer.Write( 0 );
			return;
		}

		writer.Write( _worldData.Length );
		writer.WriteArray( _worldData );
	}

	void Component.INetworkSnapshot.ReadSnapshot( ref ByteStream reader )
	{
		var length = reader.Read<int>();
		if ( length > 0 )
		{
			_worldData = reader.ReadArray<byte>( length ).ToArray();
		}
	}

	// The loading screen will wait for this Task to finish
	protected override async Task OnLoad()
	{
		if ( _worldData != null )
		{
			await GenerateMeshesAsync( _worldData );
		}
	}

	private async Task GenerateMeshesAsync( byte[] data )
	{
		// ... heavy processing ...
		await Task.Delay( 100 ); // Example delay
	}
}
```

## Troubleshooting

:::danger Stream Read/Write Mismatch
The order and exact data types you write in `WriteSnapshot` **must exactly match** what you read in `ReadSnapshot`. If you write an `int` and then a `string`, but attempt to read a `string` and then an `int`, the entire network stream for the snapshot will become corrupted, leading to disconnects and catastrophic errors.
:::

:::warning Does not sync after the initial connection
`INetworkSnapshot` is **only** called when the snapshot is generated for a joining client. If the `_voxelData` array changes *after* the client has joined, it will not automatically re-sync. You must use [RPC Messages](rpc-messages.md) or standard `[Sync]` properties for real-time updates.
:::

## Related Pages
- [Sync Properties](sync-properties.md)
- [Networked Objects](networked-objects.md)
