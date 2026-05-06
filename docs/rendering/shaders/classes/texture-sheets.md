---
title: "Texture Sheets"
icon: "✂️"
created: 2023-11-26
updated: 2024-12-08
---

# Texture Sheets

Using texture sheets in shaders is made easy using the `Sheet` class.

# What are Texture Sheets?

Sheets are textures that contain multiple images. These images are arranged into sequences. A texture sheet can have multiple sequences.


So for example, one texture sheet could include 10 different sequence of leaves, all with one frame. This would be useful in particles, to add variety without using multiple textures (and having to do 10 different draw calls). Or you could have a texture with a single sequence with 64 images of smoke, which when animated makes the smoke look like it's moving.

# Api

## Sheet::Single

Will return updated UV information for the sheet. This is generally called in the vertex shader, and the updated UVs are passed down to the pixel shader to sample.

```csharp
// Get the texture sheet data from somewhere
float4 g_SheetData < Attribute( "BaseTextureSheet" ); >;

// ...

o.uv = Sheet::Single( g_SheetData, sequenceId, g_flTime, i.uv );
```


## Sheet::Blended

Returns two uv's, with a float to blend between them. This is for animated sheets.

```csharp
// Get the texture sheet data from somewhere
float4 g_SheetData < Attribute( "BaseTextureSheet" ); >;

// vertex shader (usually)

Sheet::Blended( g_SheetData, sequenceId, g_flTime, i.uv, o.sheetUv.xy, o.sheetUv.zw, o.sheetBlend );

// pixel shader

float4 col = Tex2D( g_ColorTexture, i.sheetUv.xy );

if ( i.sheetBlend > 0 )
{
	float4 col2 = Tex2D( g_ColorTexture, i.sheetUv.zw );
	col = lerp( col, col2, i.sheetBlend );
}
```
