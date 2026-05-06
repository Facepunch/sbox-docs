---
title: "GPU Instancing"
icon: "⚡"
created: 2024-12-05
updated: 2024-12-05
---

# GPU Instancing

GPU instancing is an optimization where multiple instances of the same model with the same material can be drawn in a single draw call. This is a big optimization when rendering things that appear many times in a scene such as foliage.

## Standard Instancing

Everything is automatically instanced by default, if the renderer can batch your models into 1 draw call it will.

To get the Object→World matrix in your vertex shader you use the following function:

```cpp
float3x4 CalculateInstancingObjectToWorldMatrix( const VS_INPUT i );
```

Extra per instance data can be grabbed by using:

```cpp
ExtraShaderData_t GetExtraPerInstanceShaderData( const VS_INPUT i );
```

ExtraShaderData allows you to still have instances even though some data is different between them, currently only tint color is changeable within instances:

```cpp
struct ExtraShaderData_t
{
	float4 vTint;
	uint nBlendWeightCount;			// if D_SKINNING, blend weight count
};
```

Instances can be drawn manually in C# with: `Graphics.DrawModelInstanced( Model, Span<Transform> )` this allows you to draw 1000s of instances without burdening the scene system.

## Procedural Instancing

Procedural instancing is useful for programmatic shaders that are to be invoked from C# or indirectly.
No transforms are passed only the count of instance ids, you would derive some form of transform yourself from them within the shader.

You can get the instance id from SV_InstanceID in your vertex function.

This is also useful for indirect draw calls.

```cpp
PixelInput MainVs( VertexInput i, uint instanceID : SV_InstanceID )
{ 
    float3 offsetPosition = float3( 0, 0, 64 * instanceID );
}
```

See:

* `Graphics.DrawModelInstanced( Model, count )`
* `Graphics.DrawModelInstancedIndirect( Model, GpuBuffer )`
