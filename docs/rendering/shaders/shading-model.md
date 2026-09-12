---
title: "Shading Model"
icon: "🔦"
created: 2024-12-08
updated: 2026-09-03
---

# Shading Model

Shading Models are what process how a model is lit, this is typically the final step and output of your shader.

Consumes your [Material](/rendering/shaders/material.md) and shades the output with lighting, atmospheric, and deals with everything needed to show specific stages of rendering if [Debug Views](/rendering/shaders/modes.md) are enabled.

The traditional flow of a lit shader is `Input Interpolants > Material Definition > Output Shading Model`

## ShadingModelStandard

The default Source 2 Lighting.

```csharp
float4 MainPs(PixelInput i)  : SV_Target0
{
   Material m = Material::Init( i );
   return ShadingModelStandard::Shade( m );
}
```

### float4 ShadingModelStandard::Shade( Material m )

* Consumes the material set from `m` and does the standard lighting from it.

  Returns the final output of the shader in a `float4`

## Custom Shading Model Example

The default Shading Model should have everything you need, but if your game has a custom art style you can try to implement your own Shading Model with ease.

When implementing your own Shading Model, the best practice is to segment it by **Direct/Indirect** and each of these again in **Specular/Diffuse** lighting.

This is an example of a simple toon shader:

```csharp
class ShadingModelToon
{
    static float4 Shade( Material m )
    {
        float4 color = 0;
        
        // Direct Lighting
        for( uint i = 0; i < Light::Count( m.ScreenPosition ); i++ )
        {
            Light l = Light::From( m.WorldPosition, m.ScreenPosition, i );
            ...
            
            float3 diffuse = /* */
            float3 specular = /* */
            
            color += diffuse + specular;
        }
        
        // Indirect Lighting
        {
            float3 diffuse = 0;
            {
                diffuse = /* ambient light */
                diffuse *= min( m.AmbientOcclusion, ScreenSpaceAmbientOcclusion::Sample( m.ScreenPosition ) );
            }
            
            float3 specular = 0;
            {
                specular = /* envmap */
                specular += /* toon fresnel */
            }
            
            color += diffuse + specular;
        }
        
        if( DepthNormals::WantsDepthNormals() )
            return DepthNormals::Output( m.Normal, m.Roughness, color.a );

        if( ToolsVis::WantsToolsVis() )
        {
            // See below - build a ToolsVis from your lighting terms and let it
            // draw whichever debug view the editor has selected.
        }

        // Composite atmospherics after lighting
        color.rgb = Fog::Apply( m.WorldPosition, m.ScreenPosition.xy, color.rgb );
        
        return color;
    }
}
```

### Supporting Debug Views

`ToolsVis::WantsToolsVis()` tells you the editor is asking for a debug view rather than the normal image. To support them, initialize a `ToolsVis` with your lighting terms and call the `Handle*` methods for the ones you care about:

```csharp
ToolsVis vis = ToolsVis::Init( color, diffuse, specular, indirectDiffuse, indirectSpecular, transmissive );

vis.HandleAlbedo( color, m.Albedo );
vis.HandleNormalWs( color, m.Normal );
vis.HandleRoughness( color, float2( m.Roughness, m.Roughness ) );
// ...and so on for the views you want to support

return color;
```

`ShadingModelStandard` wires up every one of these in its own `DoToolsVis`, so [shadingmodel.hlsl](https://github.com/Facepunch/sbox-public/blob/master/game/core/shaders/common/shadingmodel.hlsl) is the reference to copy from. Note `DoToolsVis` itself takes the `LightingTerms_t` that `ShadingModelStandard::Shade` fills in from the built-in lighting path, so it isn't directly reusable from your own shading model.
