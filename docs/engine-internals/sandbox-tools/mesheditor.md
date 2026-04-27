---
title: "MeshEditor"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/MeshEditor/PrimitiveBuilder.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Tools.MeshEditor

This section covers the internal implementation of the `MeshEditor` subsystem within the s&box Editor.

## Architecture

This folder contains the C# source code that handles the editor-side functionality for MeshEditor. It is loaded only when the engine runs in tools mode (`sbox-dev.exe`).

While the core mesh editing logic (like vertex manipulation, edge loops, and UV unwrapping) happens in native code, the managed `MeshEditor` namespace provides utilities for procedural geometry generation and interacting with mesh data from C#.

## Core Concepts

The most prominent concept in the managed layer is the `PrimitiveBuilder` and its internal `PolygonMesh` representation.

### Building Primitives

Engine developers or tool creators can define new primitive shapes (like cubes, cylinders, or custom procedural meshes) by extending the `PrimitiveBuilder` abstract class. 

The builder is passed a `PolygonMesh` object, which acts as a lightweight managed representation of geometry (Vertices and Faces) before it is converted to a native Hammer mesh.

```csharp
using Editor.MeshEditor;

public class MyCustomPyramidBuilder : PrimitiveBuilder
{
    // The material to apply to this primitive
    public override Material Material { get; set; } = Material.Load( "materials/dev/reflectivity_30.vmat" );

    public override void Build( PolygonMesh mesh )
    {
        // 1. Define the vertices
        int top = mesh.AddVertex( new Vector3( 0, 0, 100 ) );
        int fl = mesh.AddVertex( new Vector3( 50, 50, 0 ) );
        int fr = mesh.AddVertex( new Vector3( 50, -50, 0 ) );
        int bl = mesh.AddVertex( new Vector3( -50, 50, 0 ) );
        int br = mesh.AddVertex( new Vector3( -50, -50, 0 ) );

        // 2. Define the faces using the vertex indices (ordered counter-clockwise)
        mesh.AddFace( top, fl, bl ); // Left face
        mesh.AddFace( top, bl, br ); // Back face
        mesh.AddFace( top, br, fr ); // Right face
        mesh.AddFace( top, fr, fl ); // Front face
        
        // Base face
        mesh.AddFace( fl, fr, br, bl );
    }

    public override void SetFromBox( BBox box )
    {
        // Setup initial properties (like scale/position) based on a bounding box drawn by the user
    }
}
```

### The `PolygonMesh` Structure

The `PolygonMesh` is optimized for rapid generation:
- **`Vertices` List:** A list of `Vector3` positions. The `AddVertex` method automatically deduplicates positions (within a small tolerance) to ensure continuous, connected geometry.
- **`Faces` List:** A list of faces, where each face is an array of indices pointing to the `Vertices` list. Faces must be wound counter-clockwise to calculate normals correctly.
- **Direct Position Insertion:** You can optionally pass `Vector3` arrays directly into `AddFace(params Vector3[])` and the `PolygonMesh` will handle vertex creation and index linking automatically.

## Common Patterns

1. **2D vs 3D Primitives:** Builders can override the `Is2D` property to true. If a primitive is 2D, the user's drag-box in the editor viewport will be constrained to have no depth, forcing a flat bounding box before `SetFromBox(BBox)` is called.