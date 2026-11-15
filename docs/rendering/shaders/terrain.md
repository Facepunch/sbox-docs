---
title: "Terrain"
icon: "⛰️"
created: 2026-09-17
updated: 2026-11-15
---

s&box comes with a terrain system and fairly large shader backend for various terrain things: meshlets, encoded control map, compact material format, structs, alpha clipping, and some other features.  

:::warning
🔨👶 Terrain system is still receiving new updates and it's probably fair to warn that this API may change over the time. If that happens, we will make sure to reflect these changes on this page.
:::

# Terrain Shader

Terrain shader lives in `/core/shaders/terrain.shader`. This is a main shader that is used on terrain by default. You can swap the material to something else (like material with a custom terrain shader) in the component settings. 

It uses its own unique vertex & pixel input structs. Vertex input holds only vertex position, LOD, and instance ID. All other mesh data like geometry normals, tangents and UVs are all sampled/calculated within a pixel shader. Pixel input holds local/world position of a terrain vertex, as well as screen position (pixel shader only) and pixel position (vertex shader only).

# Terrain Class

All of this lives in `/core/shaders/terrain/TerrainCommon.hlsl`, nothing of this will work if you don't have this file included in the shader. 

## Receiving Terrain Struct 
- `TerrainStruct Terrain::Get()`
- - Returns the struct data for terrain

## Terrain Geometry Maps
- `Texture2D Terrain::GetHeightMap()`
- - Returns a Texture2D object of terrain's heightmap.
- `Texture2D Terrain::GetControlMap()`
- - Returns a Texture2D object of terrain's indexed control map.
- `Texture2D Terrain::GetNormalMap()`
- - Returns a Texture2D object of terrain's baked geometry normals. Don't forget to remap them to [-1 to 1] range after sampling!

## Terrain Geometry Normals
- `float3 Terrain::SampleNormal( float2 uv )`
- - Returns remapped [-1 to 1] terrain's baked geometry normals as `float3`
- - `uv` must be local terrain UV coordinates
- `float3 Terrain::NormalBasis( float2 uv, out float3 tangentU, out float3 tangentV )`
- - Returns remapped [-1 to 1] terrain's baked geometry normals as `float3` AND also provides tangentU & tangentV
- - Same as above, `uv` must be local terrain UV coordinates

## Coordinates
- `float3 Terrain::WorldToLocal( float3 worldPos )`
- - Converts provided world-space position to local terrain coordinates.
- `float2 Terrain::LocalToUV( float2 localPos )`
- - Converts local terrain coordinates to proper UV coords in [0 to 1] range that can be used for sampling terrain textures
- `float2 Terrain::GetUV( float3 worldPos )`
- - Converts provided world-space position straight to terrain's UV coordinates.

## Terrain Heightmap
- `float Terrain::GetHeight( float2 localPos )`
- - Returns terrain height at given local terrain coordinates, with height scale applied. Keep in mind that returned height value is local to terrain, it doesn't reflect its actual position in world.
- `float Terrain::GetWorldHeight( float3 worldPos )`
- - Returns terrain height at given world-space position, with height scale applied. This height will be in world-space coordinates, representing the Z component, not local to terrain.
- `float Terrain::GetDistanceToSurface( float3 worldPos )`
- - Returns distance to terrain surface at given world-space position.

## Misc 
- `int Terrain::Count()`
- - Returns all terrains on scene.
- `bool Terrain::IsInBounds( float3 worldPos )`
- - Checks if provided world-space position is within terrain bounds.
- `float Terrain::GetBlendFactor( float3 worldPos, float blendLength )`
- - Get a 0-1 blend factor for mesh blending based on distance to terrain surface.
- - Returns 1 at terrain surface, fading to 0 at blendLength distance above

## Texture Sampling
See section "Sampling Terrain from Any Shader" below if you'd like to sample a full terrain material instead of color.

- `float3 Terrain::SampleMaterialColor( float2 texUV, CompactTerrainMaterial material )`
- - Returns given material's color map, must provide terrain UV coordinates.
- - Must provide an exact terrain material to sample color from it
- - You can also add a third argument `int mipLevel` to this function to get specific mip level of this material's color texture.
- `float3 Terrain::SampleColor( float3 worldPos )`
- - Samples color map of terrain from given world-space coordinates, with material blending and other necessary steps. Keep in mind that this is only color, not full terrain material.
- - Add a second argument `int mipLevel` to get exact mip level

# Sampling Terrain From Any Shader

We have a common API for sampling a complete terrain splat with all necessary steps applied: calculating weights and gathering control bits, decoding control map, fetching terrain materials and blending all materials at given pixel together. It is used by main terrain shader as well, so if you use this function from your own custom shader, it will have visuals fully identical to the terrain mesh. 

Returned data is represented as a Material struct, it holds all terrain textures except for heightmap, since we don't have a place for it in the material struct sadly.

- `Material Terrain::Sample( float3 worldPos, bool bUseGeometricNormals = false )`
- - Samples terrain splat at given world-space coordinates.
- - Please note that by default this function will return plain terrain splat without transforming them with terrain's geometry normals. Set the second argument to `true` if you want to sample them as well.
- `Material Terrain::Sample( float2 localPos, bool bUseGeometricNormals = false )`
- - Does the same thing as function above, but this one accepts **local** terrain coordinates, in case you have them in your code available already.
- `Material Terrain::Sample( float2 localPos, float2 localDdx, float2 localDdy, uint4 controlBits, float4 quadWeights, bool bUseGeometricNormals = false )`
- - Samples terrain splat at given local terrain coordinates, but here you can pass your own control bits and weights.

You can also transform tangent normals with terrain's baked normals separately:
- `void Terrain::ApplyGeometricNormals( inout Material m, float2 uv )`
- - Blends tangent normals in provided material with terrain's baked geometry normals. Returns nothing, it will iterate on given material struct.
- - Expects terrain UV coordinates. If you don't have them, you can use `Terrain::GetUV`. 
- - Under the hood it samples terrain's baked normal map and then calculates tangentU/tangentV, re-orthonormalizes to account for possible custom terrain scale, and then finally blends normals. It will also store tangentU and tangentV into Material's `WorldTangentU` and `WorldTangentV` accordingly, in case you will need them for anything else later on.

## Using on models

This API can be used in any shader, not just for custom terrain shaders, it will work with any models as well. To use it in a regular shader, you need to include `terrain/TerrainCommon.hlsl` in pixel shader (`PS` section), and then you're good to go. Here's a basic example:

```cpp
float4 MainPs( PixelInput i ) : SV_Target0
{
	Material basicMaterial = Material::Init( i.vPositionWithOffsetWs, i.vPositionSs );
    // ... setup your basic material here...
    // and then sample terrain at given world-space position, with geometric normals applied
	Material terrain = Terrain::Sample( basicMaterial.WorldPosition, true );

    // blend two materials together, very shoddy example but you get the idea
    Material final = Material::lerp( basicMaterial, terrain, 0.5f );

    return ShadingModelStandard::Shade( i, final ); 
}    
```

# Terrain Struct

Each terrain instance holds following data in its struct: 

```cpp
struct TerrainStruct
{
    float4x4 Transform;
    float4x4 TransformInv;
    int HeightMapTexture;
    int ControlMapTexture;
    float UnitsPerTexel;
    float HeightScale;
    bool HeightBlending;
    float HeightBlendSharpness;
    int samplerindex;
    int NormalMapTexture;
};
```

- `Transform` and `TransformInv` are terrain's world transform (and inverse transform) matrix, you need to use it to get proper local terrain coordinates that match its transform on scene. Transform holds terrain's position, rotation and scale.
- `HeightMapTexture`, `ControlMapTexture` and `NormalMapTexture` are bindless IDs for terrain's heightmap, control map and baked geometry normals.
- `UnitsPerTexel` controls the density of texture tiling.
- `HeightScale` is a setting that controls the total height of terrain in units. So if height scale is `1000` then it means that terrain's physical maximum height will be 1000 units
- `HeightBlending` is a bool that controls whether this terrain uses height blending for splats. If it's disabled, then simpler bilinear blending will be used instead.
- `HeightBlendSharpness` controls the sharpness of height blending between splats
- `samplerindex` is a bindless ID for sampler state, which will be used for sampling terrain textures

To avoid referring to a full buffer name every time you need to get something from terrain data, you can use `Terrain::Get()` instead. So if you want to grab terrain's height scale, you just do `Terrain::Get().HeightScale`, or `Terrain::Get().Transform` to grab terrain's world transform matrix. 

# Terrain Material

Each terrain material holds a bunch of data in its struct:

```cpp
struct TerrainMaterial
{
    int bcr_texid;
    int nho_texid;
    float uvscale;
    uint flags;
    float metalness;
    float heightstrength;
    float normalstrength;
    float displacementscale;

    bool HasFlag( TerrainFlags flag )
    {
        return (flags & flag) != 0;
    }
};
```

- `bcr_texid` is a bindless ID of material's color map texture (stored in RGB) and roughness (alpha). You need to apply `SrgbGammaToLinear` on RGB color map to get accurate visuals.
- `nho_texid` is a bindless ID of material's tangent normal map (RG channels), heightmap (blue channel) and ambient occlusion (alpha channel). Keep in mind that this texture provides only normal map's RG maps, blue channel is later reconstructed using `ComputeNormalFromRGTexture()` helper.
- `uvscale` is material's UV scale
- `flags` is used for storing various flags in provided material. Currently there's only just one terrain flag, which is `NoTile`
- `metalness` is a float value in 0-1 range indicating how metallic is this material. It isn't a texture, nor a bindless ID for it, just uniform float value.
- `heightstrength` is a float value for controlling the strength of material's heightmap texture.
- `normalstrength` is a float value for controlling the intensity of material's normal map.
- `displacementscale` is a float value for controlling the intensity of terrain displacement.
- `HasFlag( flag )` checks if current material has the provided flag. As it's stated above, currently there's only `NoTile` flag.

All terrain materials are stored in a structured buffer named `g_TerrainMaterials`. To get an exact terrain material you must know its index, one of the ways to do so is to decode control map and get base/overlay material ID from CompactTerrainMaterial struct. For more details, see the next section. 

# Encoded Terrain Control Map

Our terrain system uses encoded control map which supports up to 32 terrain maps. When texel is decoded, it provides a CompactTerrainMaterial struct. Its structure looks like this:

```cpp
struct CompactTerrainMaterial
{
    uint BaseTextureId;
    uint OverlayTextureId;
    uint BlendFactor;
    bool IsHole;
    uint Reserved;
}
```

- `BaseTexureId` points at base material's ID in global terrain materials buffer (`g_TerrainMaterials`)
- `OverlayTextureId` points at overlay material's ID in global terrain materials buffer 
- `BlendFactor` is a blend factor between base and overlay materials, where 0 = full base, and 255 = full overlay. This value must be remapped to [0 to 1] before being used in actual blending math. See below for details.
- `IsHole` is a flag that indicates that given texel is a hole
- `Reserved` is reserved (duh) and contains nothing at the moment 

So basically every encoded control map texel holds two terrain materials inside of it, along with information for alpha holes and blend factor for them, it doesn't store actual terrain material textures. So for example, to get an actual struct that holds texture IDs for base texture, you'd need to do this:

```cpp
// where 'material' is CompactTerrainMaterial
TerrainMaterial baseMat = g_TerrainMaterials[material.BaseTextureId];
```

## Decoding
Structure of encoded control map pixel looks like this:

| Bits | Purpose | Limit |
|------|---------|-------|
| 0-4 | Base texture ID | 0-31 |
| 5-9 | Overlay texture ID | 0-31 |
| 10-17 | Blend factor between base & overlay | 0-255 |
| 18 | Hole flag (1 = hole, 0 = solid) | ... |
| 19-31 | Reserved, 13 bits |

There are two ways to decode this struct and get useable data:

- Get control quads (`Terrain::GatherControlQuad`) to fetch 4 texels of bilinear quad around the UV as raw `uint` bits from terrain's control map, and then decode each bit using `CompactTerrainMaterial::Decode( uint packed )`
```cpp
CompactTerrainMaterial mat00 = CompactTerrainMaterial::Decode( controlBits.x );
CompactTerrainMaterial mat10 = CompactTerrainMaterial::Decode( controlBits.y );
CompactTerrainMaterial mat01 = CompactTerrainMaterial::Decode( controlBits.z );
CompactTerrainMaterial mat11 = CompactTerrainMaterial::Decode( controlBits.w );
```
- Alternatively, you can decode from control map stored in terrain's GPU storage using `CompactTerrainMaterial::DecodeFromFloat( float control )` without four control quads, which is what commonly used by compute shaders.


## Encoding
This has no use in live terrain sampling, but may be useful if you are interacting with terrain control map from a compute shader and want to edit it.

Once you finish editing values inside this struct, use `Encode()` to get encoded uint control map. You can also use `EncodeToFloat()` to encode it as a **float** instead of **uint**, since control map texture is stored as a float in the terrain's GPU storage.

For example, in `cs_terrain_splat` compute shader, which is used for editing terrain splat using GPU, once we're done modifying what we need, we write back to this control map at given texel like this:

```cpp
// 'ControlMap' is a RWTexture2D control map which we edit in this shader
// 'material' is a CompactTerrainMaterial struct
ControlMap[texel] = material.EncodeToFloat();
```

## Blend
Every terrain material stores blend value, but even after decoding it is represented in [0 to 255] range. Before applying blending between base and overlay materials, make sure to use `GetNormalizedBlend()` which will remap the value to [0 to 1] range. 

```cpp
CompactTerrainMaterial mat = /* <...> */;
float materialBlend = mat.GetNormalizedBlend(); // returns material blend in 0-1 range 
```

When editing control map from compute shaders, make sure to remap this value back to [0 to 255] range before writing encoded control map texel back to RW texture:

```cpp
// remapping 0-1 blend back to 0-255 range before writing control map to disk
material.BlendFactor = uint( saturate( overlayWeight ) * 255.0 + 0.5 );
```

