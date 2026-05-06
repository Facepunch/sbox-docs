---
title: "Shading Model"
icon: "🔦"
created: 2024-12-08
updated: 2024-12-16
---

# Shading Model

Shading Models are what processes how a model is lit, this is typically is the final step and output of your shader.

Consumes your [Material](/rendering/shaders/material.md) and shades the output with lighting, atmospheric, and deals with everything needed to show specific stages of rendering if [Debug Views](/rendering/shaders/modes.md) are enabled.

The traditional flow of a lit shader is `Input Interpolants > Material Definition > Output Shading Model`

## ShadingModelStandard

The default Source 2 Lighting.

```csharp
float4 MainPs(PixelInput i)  : SV_Target0
{
   Material m = Material::From( i );
   return ShadingModelStandard::Shade( m );
}
```

### float4 ShadingModelStandard::Shade( Material m )

* Consumes the material set from `m` and does the standard lighting from it.

  Returns the final output of the shader in a `float4`

## Custom Shading Model Example

The default Shading Model should have everything you need, but if your game has a custom art style you can try to implement your own Shading Model with ease.

When implementing your own Shading Model, the best practice is to segmenting it by **Direct/Indirect** and each of these again in **Specular/Diffuse** lighting.

This is an example of a simple toon shader:

```csharp
class ShadingModelToon
{
    static float4 Shade( Material m )
    {
        float4 color = 0;
        
        // Direct Lighting
        for( int i=0; i < DynamicLight::Count(); i++ )
        {
            Light l = DynamicLight::From( ScreenPosition, WorldPosition, i );
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
                diffuse *= min( m.AmbientOcclusion, ScreenSpaceAmbientOcclusion::Sample( m.ScreenPosition );
            }
            
            float3 specular = 0;
            {
                specular = /* envmap */
                specular += /* toon fresnel */
            }
            
            color += diffuse + specular;
        }
        
        if( DepthNormal::WantsDepthNormal() )
            return DepthNormal::Output( m );

        if( ToolVis::WantsToolVis() )
            return ToolVis::Output( color, m );

        // Composite atmospherics after lighting
        color.xyz = Fog::Apply( m.WorldPosition, m.ScreenPosition.xy, color );
        
        return color;
    }
}
```
