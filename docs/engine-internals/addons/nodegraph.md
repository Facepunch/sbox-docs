---
title: "Tools Addon: NodeGraph"
icon: "🌲"
sources:
  - game/addons/tools/Code/NodeGraph/
created: 2026-04-27
updated: 2026-04-27
---

# NodeGraph (Visual Scripting Editor)

The `NodeGraph` directory within the Tools addon contains the base implementation for visual node-based editors in the s&box development environment. This is the underlying framework that powers both the **ActionGraph** (visual gameplay scripting) and the **ShaderGraph** (visual shader editing).

## Quick Working Example

If you are extending the editor to create a custom node graph, you generally interact with the graph through the `GraphView` widget:

```csharp
using Editor.NodeEditor;
using Sandbox;

public class MyCustomGraphWindow : DockWindow
{
    private GraphView graphView;

    public MyCustomGraphWindow()
    {
        Title = "My Custom Graph";
        
        // Initialize the graph view
        graphView = new GraphView();
        
        // Add it to the window layout
        Layout = Layout.Column();
        Layout.Add( graphView );
    }

    public void LoadGraph( IGraph myGraphData )
    {
        // Assigning the data model to the view rebuilds the UI
        graphView.Graph = myGraphData;
    }
}
```

1.  **Rendering Custom Nodes:** The visual appearance of nodes is highly standardized. You typically do not need to subclass `NodeUI` unless you are creating a completely non-standard node shape (like a reroute or comment node).
2.  **Data Binding:** The UI classes in `NodeGraph` expect an underlying data model that implements specific interfaces (`IGraph`, `INode`, `IPlug`). Changes in the UI (like connecting two plugs) are passed down to the data model, and changes in the data model cause the UI to redraw.
3.  **Context Menus:** Right-clicking the `GraphView` canvas typically brings up a searchable `NodeQuery` menu, allowing the user to instantiate new nodes.

## Troubleshooting

:::warning Common Gotchas
- **"Ghost" Connections:** If a visual `Connection` exists but the underlying `IGraph` data model does not reflect it (or vice versa), the graph will behave unpredictably. Always ensure the UI is synchronized with the data.
- **Layout Jitter:** If you build custom inline editors (`ValueEditor`), ensure their `Layout()` method accurately computes their size. Incorrect sizes will cause the node to jitter as it tries to pack its contents.
:::

## Related Pages
- [Tools Addon Code Architecture](tools-code.md)
- [ActionGraph](../../systems/actiongraph/index.md)
- [ShaderGraph](../../systems/shader-graph/index.md)

