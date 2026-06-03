---
title: "GPU Instancing"
icon: "⚡"
created: 2024-12-05
updated: 2026-10-10
---

# GPU Instancing

GPU instancing is an optimization where multiple instances of the same model with the same material can be drawn in a single draw call. This is a big optimization when rendering things that appear many times in a scene such as foliage.

## Standard Instancing

Everything is automatically instanced by default, if the renderer can batch your models into 1 draw call, it will.

### Instance ID

Standard vertex input provides `i.nInstanceTransformID` for the instance's transform slot. `i.nBoneIndex` is a single blend index for rigid transforms; compute skinning reads its blend indices separately from the mesh.

To add batched instance ID and bone index to your custom vertex struct, add this:
```cpp
uint nInstanceTransformID : TEXCOORD13 < Semantic( InstanceTransformUv ); >;
uint nBoneIndex 		  : BLENDINDICES < Semantic( BlendIndices ); >;
```

### Helpers

* To get the Object→World matrix in your vertex shader you use the following function:

```cpp
float3x4 GetTransformMatrix( uint nTransformIndexID, uint nBlendIndex = 0 )
```

With the standard vertex input, call `GetTransformMatrix( i.nInstanceTransformID, i.nBoneIndex )` to include the bone index when needed.

* Extra per instance data can be grabbed by passing the instance's transform ID:

```cpp
ExtraShaderData_t GetExtraPerInstanceShaderData( uint nTransformIndexID );
```

In a vertex shader you'd call it like `GetExtraPerInstanceShaderData( i.nInstanceTransformID )`.

The returned data contains the instance tint and alpha, vertex-cache offset and blend-weight count:

```cpp
struct ExtraShaderData_t
{
	float4 vTint;
	uint nVertexCacheOffset;
	uint nBlendWeightCount;	// blend weight count
};
```

* There's also a helper function which allows quickly getting transform scale from object->world matrix of your instance:
```cpp
float3 ComputeMatrixScale( float3x4 m )
```

## Procedural Instancing

`Graphics.DrawModelInstanced( Model, int count )` and `Graphics.DrawModelInstancedIndirect( Model, GpuBuffer buffer, int bufferOffset = 0, RenderAttributes attributes = null )` draw instances without supplying per-instance transforms. The latter reads its draw arguments from the GPU buffer. Your shader derives or loads each transform itself.

You can get the instance id from SV_InstanceID in your vertex function. This is also useful for indirect draw calls.

```cpp
float4 MainVs( VertexInput i, uint instanceID : SV_InstanceID ) : SV_Position
{
    float3 worldPosition = i.vPositionOs + float3( 0, 0, 64 * instanceID );
    return Position3WsToPs( worldPosition );
}
```

This minimal example outputs only the clip-space position. Add the interpolants your pixel shader needs when using it in a material shader.

## Supplying Instance Transforms

Use `Graphics.DrawModelInstanced( Model, Span<Transform> transforms, RenderAttributes attributes = null )` to supply transforms from C# and use the standard shader instancing path.

For GPU-driven transforms and counts, use this overload:

```csharp
Graphics.DrawModelInstancedIndirect( model, transformBuffer, indirectArgs );
```

`transformBuffer` contains `Sandbox.Rendering.GpuInstanceTransform` elements; `indirectArgs` contains `DrawIndexedInstancedArguments` and must be created with `GpuBuffer.UsageFlags.IndirectDrawArguments`. This overload feeds the standard `GetTransformMatrix()` path, so material shaders can use the supplied transforms without implementing procedural placement.
