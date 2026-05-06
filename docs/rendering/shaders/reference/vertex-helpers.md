---
title: "Vertex Helpers"
icon: "🕸️"
created: 2024-12-11
updated: 2024-12-16
---

# Vertex Helpers

Helper functions to do standard vertex shader, does instancing, skinning, etc.

This would eventually be moved to a `Vertex` or inside `PixelInput` class that would be more malleable.

## Reference

* `ProcessVertex( VertexInput i )`
  * Sets up common processing for the Vertex Shader, processing the following steps:
    * Instancing
    * Skinning
    * Position on the world
    * Normals, Tangents from transformation
* `FinalizeVertex( PixelInput o )`
  * Does post-processing for the vertex, converts the position on the world to what's displayed on the screen ( Projection Space )
  * \

You're expected to commonly use these on start and end of the Vertex Block, and have any modifications of it in between them

```cpp
VS
{
    #include "common/vertex.hlsl"
    //
    // Main
    //
    PixelInput MainVs( VertexInput i)
    {
        PixelInput o = ProcessVertex(i);
        // Add your vertex manipulation functions here
        return FinalizeVertex(o);
    }
}
```
