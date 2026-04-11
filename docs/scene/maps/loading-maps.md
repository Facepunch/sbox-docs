---
title: "Loading Maps"
icon: "📂"
created: 2026-04-11
updated: 2026-04-11
---

# Loading Maps

The `MapInstance` component is what you use to load a map into your scene. Add it to any GameObject, set the map, and it handles the rest - loading geometry, creating physics, and spawning entities.

```csharp
// Get the MapInstance already in the scene
var mapInstance = Scene.GetAllComponents<MapInstance>().First();

// Check if the map is loaded
if ( mapInstance.IsLoaded )
{
    Log.Info( $"Map bounds: {mapInstance.Bounds}" );
}
```

# Setting the Map

There are three ways to specify which map to load.

## In the Editor

Select the GameObject with the `MapInstance` component and use the **Map** property dropdown in the inspector. This lets you browse `.vmap` assets from your project and any mounted packages.

## By Package Ident

Set `MapName` to a package identifier from asset.party. The map package will be downloaded and mounted automatically.

```csharp
var mapInstance = GetComponent<MapInstance>();
mapInstance.MapName = "facepunch.flatgrass";
```

## From Launch Arguments

Enable `UseMapFromLaunch` on the component. When the game starts, it will use whatever map was passed via launch arguments instead of the one set in the editor.

```csharp
// The map from launch arguments is accessible here
var launchMap = LaunchArguments.Map;
```

This is useful for dedicated servers or lobbies where the map is chosen dynamically.


# Loading and Unloading

Map loading is asynchronous. The component starts loading when it is enabled, and will show a loading screen while the map data is being prepared.

:::info
Internally, `MapInstance` uses the `OnLoad` component method. The loading screen stays open until all component `OnLoad` tasks complete.

:::

If you change `MapName` at runtime, the component will automatically unload the current map and load the new one.

```csharp
var mapInstance = GetComponent<MapInstance>();

// Swap to a new map at runtime
mapInstance.MapName = "facepunch.pool_day";
```

You can also manually unload a map.

```csharp
mapInstance.UnloadMap();
```

# Callbacks

`MapInstance` provides two callbacks to let you react to map loading state changes.

```csharp
var mapInstance = GetComponent<MapInstance>();

mapInstance.OnMapLoaded += () =>
{
    Log.Info( "Map finished loading!" );
};

mapInstance.OnMapUnloaded += () =>
{
    Log.Info( "Map was unloaded" );
};
```


# Map Bounds

Once a map is loaded you can query its world-space bounding box.

```csharp
var mapInstance = GetComponent<MapInstance>();

if ( mapInstance.IsLoaded )
{
    BBox bounds = mapInstance.Bounds;
    Log.Info( $"Map size: {bounds.Size}" );
}
```


# Multiple Maps

You can have multiple `MapInstance` components in a scene, each loading a different map. This is useful for things like 3D skyboxes or loading sub-levels.

:::warning
A `MapInstance` loaded from a scene map will skip creating nested `MapInstance` components to avoid infinite loading loops.

:::