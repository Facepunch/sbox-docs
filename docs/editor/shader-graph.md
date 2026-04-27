---
title: "Shader Graph"
icon: "🌈"
sources:
  - game/editor/ShaderGraph/Code/
updated: 2026-04-25
created: 2026-04-27
---

# Shader Graph

Shader Graph is s&box's visual node-based editor for creating custom materials and shaders without writing HLSL code.

1. **Creating a Shader:** In the Asset Browser, right-click and select `Create -> Shader Graph`. Double-click the new `.shdrgrph` file to open the editor.
2. **The Result Node:** Every graph has a final "Result" node (which varies depending on if you are making an opaque, transparent, or unlit shader). You connect the output of your node math into the pins of this Result node.
3. **Exposing Properties:** You can create variables (like "Base Color" or "Texture") and expose them to the Material Editor. This allows artists to create multiple Materials that use the same Shader Graph but have different colors or textures assigned.

## Working Code Example (Conceptual)

While Shader Graph is visual, here is the conceptual flow you might build:
1. Create a `Texture2D` node.
2. Create a `Color` node.
3. Create a `Multiply` node.
4. Connect the output of the Texture and the Color into the Multiply node.
5. Connect the output of the Multiply node into the `Albedo` pin of the Result node.

This creates a shader where a texture is tinted by a color.

## Troubleshooting

:::warning "My material is pink/black checkered!"
This indicates a shader compilation error. Open the Shader Graph and look for red error nodes or check the engine console for HLSL compilation errors. Often, this happens if you plug a Vector4 output into a Vector2 input without explicitly unpacking it.
:::

:::warning "My exposed properties don't show up in the Material Editor!"
Ensure that the variable you created in Shader Graph has the "Expose" checkbox ticked in its property settings.
:::
