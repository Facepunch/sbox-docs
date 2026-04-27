---
title: "Virtual Grid"
icon: "🔲"
sources:
  - engine/Sandbox.Engine/Systems/UI/VirtualLayouts/VirtualGrid.cs
  - engine/Sandbox.Engine/Systems/UI/VirtualLayouts/BaseVirtualPanel.cs
updated: 2026-04-25
created: 2026-04-27
---

# Virtual Grid

> **New to UI?** Reference for the `VirtualGrid` component; UI overview is at **[UI](index.md)**.

`VirtualGrid` is a highly optimized UI panel that dynamically loads and unloads its contents as you scroll, allowing you to display thousands of items without dropping frames.

## Quick Working Example

```csharp
@using Sandbox;
@using Sandbox.UI;
@inherits PanelComponent

<root style="width: 100%; height: 100%;">
	<VirtualGrid Items=@MyItems ItemSize=@(new Vector2( 120, 120 ))>
		<Item Context="item">
			<!-- This code runs only for items currently visible on screen -->
			<div class="card">
				<label>@item.Name</label>
			</div>
		</Item>
	</VirtualGrid>
</root>

@code
{
	// A massive list that would normally crash a regular UI
	public List<MyData> MyItems { get; set; } = new();

	protected override void OnStart()
	{
		for ( int i = 0; i < 10000; i++ )
		{
			MyItems.Add( new MyData { Name = $"Item {i}" } );
		}
	}
	
	public class MyData 
	{
		public string Name { get; set; }
	}
}
```

### Modifying Items

You can add or remove items from the data list at runtime. Because the UI is virtualized, you should use the methods provided by the grid (if accessing it via C#) or ensure your state management triggers a rebuild. 

If you are using a standard `List<T>` bound to `Items`, the grid will detect changes if the list instance is replaced or if the count changes.

```csharp
public void AddNewItem()
{
	MyItems.Add( new MyData { Name = "New Item" } );
	// VirtualGrid will detect the list count change and update automatically
}
```

### Passing Context into Items

The `<Item>` tag defines the Razor template for a single cell. By default, it sets the current data object to a variable named `context`, but you can rename this using the `Context` attribute for better readability.

```csharp
<VirtualGrid Items=@Players ItemSize=@(new Vector2(200, 50))>
	<Item Context="player">
		<div class="player-row">
			<img src=@player.AvatarUrl />
			<label>@player.DisplayName</label>
		</div>
	</Item>
</VirtualGrid>
```

## Configuration

| Property | Description |
|---|---|
| `Items` | An `IEnumerable<object>` containing the data you want to display. |
| `ItemSize` | A `Vector2` defining the width and height of each grid cell. |
| `Item` | The Razor template (`RenderFragment<object>`) to render inside each cell. |
| `OnCreateCell` | An `Action<Panel, object>` called when a cell is freshly instantiated. |
| `OnLastCell` | An `Action` fired when the user scrolls to view the very last item. Useful for infinite scrolling/pagination. |

## Troubleshooting

:::danger "The grid doesn't show up or looks squished!"
`VirtualGrid` must have a defined size in your styles to work correctly, because it needs to know how large its "viewing window" is to calculate how many items to render. Make sure to set `width: 100%; height: 100%;` (or a specific pixel value) on the grid element.
:::

:::warning "Gaps aren't appearing between my items!"
`VirtualGrid` natively supports CSS gap properties. Apply `row-gap` and `column-gap` (or just `gap`) directly to the grid in your CSS, rather than adding margins to the individual items.
:::

## Related Pages
- [Razor Panels](razor-panels/index.md)
