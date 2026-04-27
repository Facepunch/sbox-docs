---
title: "Undo System"
icon: "↩️"
created: 2025-01-23
updated: 2025-06-15
---

# Undo System

# Scope Based Undo

The scope based system works by creating a snapshot of a change set when the scope is entered and another one when the scope is disposed of. The system will automatically take care of restoring the state on undo/redo.

A basic **blank** scope can be created as follows:

```csharp
// In Game & Editor Code
var undoScope = Scene.Editor?.UndoScope( "You Action Name");

// In Editor Code
var undoScope = SceneEditorSession.Active.UndoScope( "Your Action Name" );
```

```csharp
// Push() will turn the scope into an disposable
using ( SceneEditorSession.Active.UndoScope( "You Action Name" ).Push() )
{
  // Actions that modify the scene
}

var undoScope = SceneEditorSession.Active.UndoScope( "You Action Name" );
using ( undoScope.Push() )
{
  // Actions that modify the scene
}

{
  using var undoScope = SceneEditorSession.Active.UndoScope( "You Action Name" ).Push();
  // Actions that modify the scene
}
```

**However, these scopes will not capture anything yet.**

You will have to **tell the scope what objects you are about to modify**.
To ensure the best performance you should keep the set of captured objects as small as possible.

## GameObjects

To capture GameObject changes use `undoScope.WithGameObjectChanges()`.
You also have to specify what part of the GameObject(s) you you would like to capture.

* `GameObjectUndoFlags.Properties`
Is always enabled and captures basic properties of the object (Parent, Transform, Name…)
* `GameObjectUndoFlags.Components`
Captures the components list of the object
* `GameObjectUndoFlags.Children`
Captures all children of the object, this can become very expensive for complex scene hierarchies, so use it only if you have to.
* `GameObjectUndoFlags.All`
Shortcut to capture everything

  \

```csharp
using var undoScope = SceneEditorSession.Active.UndoScope( "You Action Name" )
  .WithGameObjectChanges( gameObject, GameObjectUndoFlags.Properties | GameObjectUndoFlags.Components)
  .Push();
```

To capture GameObject creation you can use `GameObjectCreations()`.

```csharp
using ( SceneEditorSession.Active.UndoScope( "Create Empty" ).WithGameObjectCreations().Push() )
{
	var go = new GameObject( true, "Object" );
}
```

Similarly you can capture objects that are about to be destroyed.

```csharp
using ( SceneEditorSession.Active.UndoScope( "Delete Object(s)" ).WithGameObjectDestructions( selectedGos ).Push() )
{
	foreach ( var go in selectedGos )
	{
		if ( !go.IsDeletable() )
			return;

		go.Destroy();
	}
}
```

## Components

Components offer similar functionality. Components are always captured as a whole so there are no flags that need to be specified.

```csharp
using ( SceneEditorSession.Active.UndoScope( "Drop Material" ).WithComponentChanges( c as Component ).Push() )
{
	c.SetMaterial( material, trace.Triangle );
}
```

### Capture Creation

```csharp
using ( SceneEditorSession.Active.UndoScope( "Add Component(s)" ).WithComponentCreations().Push() )
{
    var component = go.Components.Create( componentType );
    createdComponents.Add( component );
}
```

### Capture Destruction

```csharp
using ( SceneEditorSession.Active.UndoScope( $"Cut Component" ).WithComponentDestructions( component ).Push() )
{
	component.CopyToClipboard();
	component.Destroy();
}
```

## Selections

Selections are always captured and restored on undo/redo

## Chaining

As you may have already noticed you can chain the different functions together to capture a variety of changes and events.

```csharp
var undoScope = SceneEditorSession.Active.UndoScope( "Extract Faces" )
                  .WithComponentChanges( components )
                  .WithGameObjectDestructions( gameObjects )
                  .WithGameObjectCreations();

using ( undoScope.Push() )
```

## More Examples

### Actions that span multiple frames

If you have an action that spans multiple frames (e.g. dragging something around) you can use the following pattern to create an undo.

```csharp
public class BoxColliderTool : EditorTool<BoxCollider>
{
	private IDisposable _componentUndoScope;

	public override void OnUpdate()
	{
		var boxCollider = GetSelectedComponent<BoxCollider>();
		if ( boxCollider == null )
			return;

		var currentBox = BBox.FromPositionAndSize( boxCollider.Center, boxCollider.Scale );

		using ( Gizmo.Scope( "Box Collider Editor", boxCollider.WorldTransform ) )
		{
			if ( Gizmo.Control.BoundingBox( "Bounds", currentBox, out var newBox ) )
			{
				if ( _componentUndoScope == null )
				{
                   // Create scope if it not exists
					_componentUndoScope = SceneEditorSession.Active.UndoScope( "Resize Box Collider" )
                                            .WithComponentChanges( boxCollider )
                                            .Push();
				}
				boxCollider.Center = newBox.Center;
				boxCollider.Scale = newBox.Size;
			}
   
            // Dispose scope when mouse is release
			if ( Gizmo.WasLeftMouseReleased )
			{
				_componentUndoScope?.Dispose();
				_componentUndoScope = null;
			}
		}
	}
}
```

### Group GameObjects Action

```csharp
var undoScope = SceneEditorSession.Active.UndoScope( "Group Objects" )
                  .WithGameObjectChanges( selection, GameObjectUndoFlags.Properties )
                  .WithGameObjectCreations();

using ( undoScope.Push() )
{
	var go = new GameObject();
	go.WorldTransform = first.WorldTransform;
	go.MakeNameUnique();

	first.AddSibling( go, false );

	for ( var i = 0; i < selection.Length; i++ )
	{
		selection[i].SetParent( go, true );
	}

	EditorScene.Selection.Clear();
	EditorScene.Selection.Add( go );
}
```
