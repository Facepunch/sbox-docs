---
title: "C# Method Nodes"
icon: "⌨️"
created: 2024-01-23
updated: 2024-10-09
---

# C# Method Nodes

Making a custom node in C# is as easy as writing a static method with a special *\[ActionGraphNode( id )\]* attribute. You should also add *\[Pure\]* if you want your node to be an expression node - a node without signals.


```csharp
/// <summary>
/// Increments the value by 1.
/// </summary>
[ActionGraphNode( "example.addone" ), Pure]
[Title( "Add One" ), Group( "Examples" ), Icon( "exposure_plus_1" )]
public static int AddOne( int value )
{
	return value + 1;
}

/// <summary>
/// Waits for one second.
/// </summary>
[ActionGraphNode( "example.waitone" )]
[Title( "Wait One" ), Group( "Examples" ), Icon( "hourglass_bottom" )]
public static async Task WaitOne()
{
	await Task.Delay( TimeSpan.FromSeconds( 1d ) );
}
```


![Our custom "Add One" and "Wait One" nodes in the ActionGraph editor.](./images/our-custom-add-one-and-wait-one-nodes-in-the-actiongraph-edi.png)
