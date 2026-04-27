---
title: "Render Attributes"
icon: "📥"
sources:
  - engine/Sandbox.Engine/Systems/Render/RenderAttributes.cs
created: 2026-04-27
updated: 2026-04-27
---

# Render Attributes

Render Attributes are containers that hold properties (like colors, textures, and values) which are passed from your C# code directly to the GPU for your shaders to use.

## Quick Working Example

```csharp
using Sandbox;

public class GlowingObject : Component
{
    [Property] public Color GlowColor { get; set; } = Color.Cyan;
    [Property] public Texture CustomNoise { get; set; }

    protected override void OnUpdate()
    {
        var renderer = Components.Get<ModelRenderer>();
        if ( !renderer.IsValid() ) return;

        // Set attributes on the specific SceneObject
        renderer.SceneObject.Attributes.Set( "GlowColor", GlowColor );
        renderer.SceneObject.Attributes.Set( "NoiseTexture", CustomNoise );
    }
}
```

### Setting Simple Variables

You can set basic types like `float`, `int`, `bool`, `Vector2`, `Vector3`, `Vector4`, `Matrix`, and `Color` using the `.Set()` method.

```csharp
var attrs = mySceneObject.Attributes;

// In HLSL: float Intensity;
attrs.Set( "Intensity", 5.0f );

// In HLSL: float3 TargetPosition;
attrs.Set( "TargetPosition", new Vector3( 0, 0, 10 ) );
```

### Setting Textures

If your shader has a `Texture2D` variable, you can pass a loaded C# `Texture` object to it.

```csharp
// In HLSL: Texture2D MyCustomTexture < Attribute("MyCustomTexture"); >;
attrs.Set( "MyCustomTexture", myTexture );
```

### Setting Combos

Shader combos (often used for `#if` or `#ifdef` logic in HLSL) can be toggled using `SetCombo()`.

```csharp
// Enables a combo named "D_ENABLE_GLOW"
attrs.SetCombo( "D_ENABLE_GLOW", true );

// Set an enum-based combo (e.g. D_QUALITY 0, 1, or 2)
attrs.SetCombo( "D_QUALITY", 2 );
```

### Sending Data Buffers

For Compute Shaders or complex materials, you can send arrays of struct data using `SetData()`.

```csharp
public struct ParticleData
{
    public Vector3 Position;
    public Vector3 Velocity;
}

// Ensure the array of data is sent to a StructuredBuffer in HLSL
attrs.SetData( "ParticleBuffer", myParticleArray );
```

## Troubleshooting

:::danger "My attribute isn't affecting the shader!"
1. Ensure the attribute name string in C# exactly matches the name defined in the shader code.
2. Ensure you are setting the attribute on the correct `RenderAttributes` object (e.g., if it's a post-processing effect, set it on the Camera, not the SceneObject).
3. Ensure your shader variable has `< Attribute("YourName"); >` defined in HLSL.
:::

## Related Pages
- [Attributes and Variables](attributes-and-variables.md)
- [Command Lists](command-lists.md)
- [Shader Graph Variables](../shader-graph/variables.md)
