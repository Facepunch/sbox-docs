---
title: "UI Drag and Drop"
icon: "⋮"
sources:
  - engine/Sandbox.Engine/Systems/UI/Panel/Panel.Drag.cs
  - engine/Sandbox.Engine/Systems/UI/Panel/Event/DragEvent.cs
created: 2026-04-27
updated: 2026-04-27
---

# UI Drag and Drop

How to make UI elements draggable and allow them to be dropped onto other UI elements, such as for an inventory system.

## Quick Working Example

```csharp
using Sandbox.UI;
using Sandbox;

// The item being dragged
public class DraggableItem : Panel
{
	public DraggableItem()
	{
		Style.Width = 64;
		Style.Height = 64;
		Style.BackgroundColor = Color.Red;
	}

	protected override void OnDragStart( DragEvent e )
	{
		// Tell the UI system we are dragging this panel
		e.StopPropagation();

		// You can create a temporary "ghost" panel here if you want it to follow the cursor
		Log.Info( "Drag Started!" );
	}

	protected override void OnDrag( DragEvent e )
	{
		e.StopPropagation();

		// Update position of the visual ghost panel if you have one
		// GhostPanel.Position = e.ScreenPosition;
	}

	protected override void OnDragEnd( DragEvent e )
	{
		e.StopPropagation();
		Log.Info( "Drag Ended!" );
		
		// Clean up your ghost panel here
	}
}

// The slot that can receive the dropped item
public class DropSlot : Panel
{
	public DropSlot()
	{
		Style.Width = 80;
		Style.Height = 80;
		Style.BackgroundColor = Color.Gray;
	}

	protected override void OnDragEnter( PanelEvent e )
	{
		// Highlight the slot when dragging something over it
		Style.BackgroundColor = Color.Green;
	}

	protected override void OnDragLeave( PanelEvent e )
	{
		// Remove highlight when dragging away
		Style.BackgroundColor = Color.Gray;
	}

	protected override void OnDrop( PanelEvent e )
	{
		Log.Info( "Something was dropped here!" );
		Style.BackgroundColor = Color.Gray;
		
		// Here you would typically check what is being dragged and snap it into this slot
	}
}
```

### Stopping Propagation

When you handle a drag event, you should almost always call `e.StopPropagation()`. If you don't, the event will bubble up to the parent panel, which might also try to drag itself (like a scrollable container).

### Passing Data During Drag

A common pattern is to set a static or accessible variable when a drag starts so the drop target knows *what* was dropped.

```csharp
public static DraggableItem CurrentDraggedItem;

protected override void OnDragStart( DragEvent e )
{
	CurrentDraggedItem = this;
	e.StopPropagation();
}

protected override void OnDragEnd( DragEvent e )
{
	CurrentDraggedItem = null;
	e.StopPropagation();
}
```

Then, in your Drop Target:

```csharp
protected override void OnDrop( PanelEvent e )
{
	if ( DraggableItem.CurrentDraggedItem != null )
	{
		// Move the dragged item into this slot
		DraggableItem.CurrentDraggedItem.Parent = this;
	}
}
```

## Configuration

The `DragEvent` object passed to `OnDragStart`, `OnDrag`, and `OnDragEnd` contains useful positional data:

| Property | Description |
|---|---|
| `e.ScreenPosition` | The current absolute screen position of the mouse. |
| `e.LocalPosition` | The mouse position relative to the dragged panel. |
| `e.ScreenGrabPosition` | The absolute screen position where the drag originally started. |
| `e.MouseDelta` | How much the mouse has moved since the last frame. |

## Troubleshooting

:::danger Scrolling instead of Dragging
If your panel is inside a scrollable container (like an inventory list), the container might steal the drag event to scroll itself. Ensure you call `e.StopPropagation()` inside `OnDragStart` on your draggable item to prevent the scroll view from intercepting it.
:::

## Related Pages
- [UI Concepts](../systems/ui/index.md)
