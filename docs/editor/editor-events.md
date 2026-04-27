---
title: "Editor Events"
icon: "📅"
created: 2024-12-17
updated: 2025-10-05
---

# Editor Events

Editor Events are events that are broadcast globally throughout the editor and can be listened to/fired from any [Editor Project](editor-project.md). These are useful for creating your own custom Editor Tools and making sure they can work in tandem with existing systems.

# Hooking into an EditorEvent

Hooking into an Editor Event allows you to run additional code whenever an event is called. You can control the order at which event hooks are triggered via the `Priority` variable. Events with a lower Priority run first.

```csharp
// Hooking into a named event
[Event( "scene.stop", Priority = 100 )]
void OnSceneStop()
{
  Log.Info( "The scene has stopped" );
}

// Shorthands for frequently used events
[EditorEvent.Hotload]
void OnHotload()
{
  Log.Info( "You've hotloaded!" );
}
```

Just make sure that you have the same arguments that the EditorEvent is looking for. See the table below.

# Calling a Custom EditorEvent

```csharp
// Calling an event with 0-3 arguments
EditorEvent.Run( "customevent.test" ); // No arguments
EditorEvent.Run( "customevent.add", 1, 2); // 2 arguments

// Calling an event via an interface (theoretically unlimited arguments)
// This will run the event on any Editor Widget that implements the custom interface
EditorEvent.RunInterface<ICustomEvent>( x => x.MyCustomEvent(1,2,3,4,5) );
```

# Event Interfaces

While string-based events still exist, we prefer to use event interfaces nowadays. We find that they're more stable than strings. It's more obvious if things are using it, and it's really IntelliSense friendly.

To use the interface, you just implement it. They're coded in a way that means you don't have to implement each member, so you can just implement what you want to listen to.

```csharp
public class MyCustomWidget : Widget, ResourceLibrary.IEventListener
{
	void ResourceLibrary.IEventListener.OnRegister( GameResource resource )
	{
		Log.Info( $"{resource} has been registered!" );
	}
}
```

:::tip
For widgets event listeners will register automatically, if you want to listen to events on a non widget class you will manually need to call `EditorEvent.Register( myCustomListenerInstance );`

:::

# AssetSystem.IEventListener

```csharp

/// <summary>
/// An asset has been modified
/// </summary>
void OnAssetChanged( Asset asset ) { }

/// <summary>
/// The thumbnail for an asset has been updated
/// </summary>
void OnAssetThumbGenerated( Asset asset ) { }

/// <summary>
/// Changes have been detected in the asset system. We won't tell you what, but
/// you probably need to update the asset list or something.
/// </summary>
void OnAssetSystemChanges() { }

/// <summary>
/// Called when a new tag has been added to the asset system.
/// </summary>
void OnAssetTagsChanged() { }
```

# ResourceLibrary.IEventListener

```csharp
/// <summary>
/// Called when a new resource has been registered
/// </summary>
void OnRegister( GameResource resource ) { }

/// <summary>
/// Called when a previously known resource has been unregistered
/// </summary>
void OnUnregister( GameResource resource ) { }

/// <summary>
/// Called when the source file of a known resource has been externally modified on disk
/// </summary>
void OnExternalChanges( GameResource resource ) { }

/// <summary>
/// Called when the source file of a known resource has been externally modified on disk
/// and after it has been fully loaded (after post load is called)
/// </summary>
void OnExternalChangesPostLoad( GameResource resource ) { }
```

# Other Default EditorEvents

### Editor

| Event | Arguments | Invokes |
|-------|-----------|---------|
| `editor.created` | EditorMainWindow | When the editor has just started |
| `tool.frame` |           | Every frame |
| `hotloaded` |           | When a hotload occurs |
| `refresh` |           | When assemblies are changed |
| `tools.gamedata.refresh` |           | When assemblies are changed. Runs after `refresh` |
| `app.exit` |           | When the editor is shutting down |
| `localaddons.changed` |           | When Project Settings are updated |
| `keybinds.update` |           | When Editor Keybinds have been changed |

### Asset System

| Event | Arguments | Invokes |
|-------|-----------|---------|
| `assetsystem.newfolder` |           | When a new folder was created |
| `assetsystem.openpicker` | AssetPickerParameters | When opening an Asset Picker |
| `assetsystem.highlight` | string    | When you click "Show in Asset Browser" |
| `asset.contextmenu` | AssetContextMenu | When an asset is right clicked |
| `asset.nativecontextmenu` | FolderContextMenu | When a folder is right clicked |
| `content.changed` | string    | When a file has changed in the "Content" path |
| `compile.shader` | string    | When a shader starts compiling |
| `open.shader` | string    | When opening a shader |
| `package.changed` | Package   | When you update a package |
| `package.changed.installed` | Package   | When a package is installed |
| `package.changed.uninstalled` | Package   | When a package is uninstalled |
| `package.changed.favourite` | Package   | When you favourite a package |
| `package.changed.rating` | Package   | When you upvote/downvote |

### Scenes

| Event | Arguments | Invokes |
|-------|-----------|---------|
| `scene.open` |           | When a Scene or Prefab is opened |
| `scene.startplay` |           | When you click the Play button |
| `scene.play` |           | When the Scene enters Play Mode |
| `scene.stop` |           | When the Scene exits Play Mode |
| `scene.session.save` |           | Every second a Scene is open |
| `scene.saved` | Scene     | When a Scene is saved |

### Widgets

| Event | Arguments | Invokes |
|-------|-----------|---------|
| `paintoverlay` |           | When highlighting a Panel in the "UI Panels" tab |
| `qt.mousepressed` |           | When the Editor receives a mouse event |
| `gameframe.statusbar` | StatusBar | When the status bar is being built<br>(Used to add your own Widgets) |
| `tools.headerbar.build` | HeadBarEvent | When the header bar is build built<br>(Used to add your own Widgets) |
| `editor.preferences` | NavigationView | When the preferences widget is opened<br>(Used to add your own pages) |

### Tools

| Event | Arguments | Invokes |
|-------|-----------|---------|
| `modeldoc.menu.tools` | Menu      | When launching ModelDoc |
| `hammer.initialized` |           | When hammer is opened |
| `hammer.selection.changed` |           | When hammer selection has changed |
| `hammer.rendermapview` | MapView   | For each MapView before rendering begins |
| `hammer.rendermapviewhud` |           | When the hammer hud is rendered |
| `hammer.mapview.contextmenu` | Menu, MapView | When the MapView is right clicked |
| `actiongraph.saving` | ActionGraph, GameResource | Right before an ActionGraph is saved |
| `actiongraph.saved` | ActionGraph | When an ActionGraph is saved |
| `actiongraph.inspect` | IMessageContext | When inspecting anything in the ActionGraph |
| `actiongraph.findreflectionnodes` | FindReflectionNodeTypesEvent | When attempting to get a list of reflection nodes |
| `actiongraph.findtarget` | FindGraphTargetEvent | When attempting to find the target |
| `actiongraph.globalnodes` | GetGlobalNodeTypesEvent | When attempting to get global nodes |
| `actiongraph.localnodes` | GetLocalNodeTypesEvent | When attempting to get local nodes |
| `actiongraph.querynodes` | QueryNodeTypesEvent | When filtering through an existing list of nodes |
| `actiongraph.nodemenu` | PopulateNodeMenuEvent | When populating the node menu |
| `actiongraph.createsubgraphmenu` | PopulateCreateSubGraphMenuEvent | When right clicking to create a sub-graph |
| `actiongraph.outputplugmenu` | PopulateOutputPlugMenuEvent | When clicking and dragging out of an output plug |
| `actiongraph.inputplugmenu` | PopulateInputPlugMenuEvent | When clicking and dragging out of an input plug |
| `actiongraph.gotoplugsource` | GoToPlugSourceEvent | When double clicking on a plug to go to it's source |
| `actiongraph.inputlabel` | BuildInputLabelEvent | When building an input label |
| `actiongraph.geteditorproperties` | GetEditorPropertiesEvent | Called when launching ActionGraph |
