---
title: "C# Shader Nodes"
icon: "⌨️"
created: 2025-08-20
updated: 2025-08-20
---

# C# Shader Nodes

Just like Subgraphs, you can also create your own custom Nodes with C# (and shader code) to be used in Shader Graphs.

# Creating a ShaderNode

Custom Nodes must inherit from the `ShaderNode` class, and supports \[Title\], \[Description\], \[Category\], and \[Icon\] attributes.

```csharp
[Title("My Custom Node"), Category("Custom"), Icon("favorite")]
public class MyCustomNode : ShaderNode
{
    // ...
}
```

# Defining Inputs

Inputs are `NodeInput` properties decorated with the `[Input]` attribute to define the expected input type.

```csharp

public class MyMathNode : ShaderNode
{
    [Input(typeof(float))]
    [Title("Value A")]
    public NodeInput InputA { get; set; }
    
    [Input(typeof(Vector3))]
    [Title("Direction")]
    public NodeInput Direction { get; set; }
    
    // The default value for InputA, which will be exposed in the Properties tab
    [InputDefault(nameof(InputA))]
    public float DefaultA { get; set; } = 1.0f;
}
```

Most of your typical Property Attributes can be used here such as `[Title]`, `[Range]`, and even `[ShowIf]`

# Defining Outputs

Outputs are `NodeResult.Func` properties with the `[Output]` attribute to define the expected output type.

```csharp

public class MyMathNode : ShaderNode
{
    // ...

    [Output(typeof(float))]
    [Title("Result")]
    public NodeResult.Func Result => (GraphCompiler compiler) =>
    {
        var a = compiler.Result(InputA, DefaultA);
        var dir = compiler.Result(Direction, Vector3.Forward);
        
        // Generate shader code
        return new NodeResult(1, $"dot({a}, {dir}.x)");
    };
}
```

# Dynamic Inputs/Outputs

Sometimes you'll want a variable amount of Outputs given the Inputs, or even a variable amount of Inputs given changes made in a node's Properties tab.

```csharp

public class DynamicNode : ShaderNode
{
    [Hide]
    private List<IPlugIn> _dynamicInputs = new();
    
    [Hide]
    private List<IPlugOut> _dynamicOutputs = new();
    
    public override IEnumerable<IPlugIn> Inputs => _dynamicInputs;
    public override IEnumerable<IPlugOut> Outputs => _dynamicOutputs;
    
    // You can modify these lists however you want, whenever you want.
    // This is just a basic example of creating a new plug.
    public void AddInput(string name, Type type)
    {
        var info = new PlugInfo()
        {
            Name = name,
            Type = type,
            DisplayInfo = new DisplayInfo { Name = name }
        };
        _dynamicInputs.Add(new BasePlugIn(this, info, type));
    }
}
```

# Error Handling

Implement `IErroringNode` to throw custom error messages from your Node when you've detected any invalid uses of it.

```csharp

public class ValidatedNode : ShaderNode, IErroringNode
{
    [Input(typeof(float))]
    public NodeInput Value { get; set; }
    
    // From IErroringNode
    public List<string> GetErrors()
    {
        var errors = new List<string>();
        
        if (!Value.IsValid)
        {
            // Throw an error so the shader cannot compile until we get a valid input
            errors.Add("Value input is required");
        }
        
        return errors;
    }
}
```

# Custom Node Rendering

You can override `OnPaint` to draw whatever you want on your custom node (like the built-in Binary, Unary and Texture nodes)

```csharp
public class VisualNode : ShaderNode
{
    public override void OnPaint(Rect rect)
    {
        // Custom node rendering
        Paint.SetPen(Color.White);
        Paint.SetFont("Arial", 12);
        Paint.DrawText(rect, "Custom");
    }

}
```
