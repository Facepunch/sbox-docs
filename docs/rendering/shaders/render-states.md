---
title: "Render States"
icon: "⚙️"
created: 2025-08-11
updated: 2025-08-11
---

# Render States

You can control render states in your shaders using this syntax anywhere outside of functions.

```cpp
RenderState( BlendEnable, true );
```

| Name | Description | Values |
|------|-------------|--------|
| **FillMode** | Determines how polygons are rendered (solid faces or wireframe edges). | `WIREFRAME`, `SOLID` |
| **CullMode** | Specifies which faces (front, back, or none) are culled from rendering. | `NONE`, `BACK`, `FRONT` |
| **DepthBias** | Enables or disables depth bias to adjust depth values for rendering order. | `true`, `false` |
| **DepthBiasClamp** | Enables or disables clamping of depth bias to a set range. | `true`, `false` |
| **SlopeScaleDepthBias** | Enables or disables slope-scaled depth bias for polygons. | `true`, `false` |
| **DepthClipEnable** | Enables or disables clipping of geometry against the near/far depth planes. | `true`, `false` |
| **MultisampleEnable** | Enables or disables multisample anti-aliasing. | `true`, `false` |
| **SampleMask** | Controls which samples in a multisample target are updated. | `true`, `false` |
| **DepthEnable** | Enables or disables depth testing. | `true`, `false` |
| **DepthWriteEnable** | Enables or disables writing to the depth buffer. | `true`, `false` |
| **DepthFunc** | Comparison function for depth testing. | `NEVER`, `LESS`, `EQUAL`, `LESS_EQUAL`, `GREATER`, `NOT_EQUAL`, `GREATER_EQUAL`, `ALWAYS` |
| **StencilEnable** | Enables or disables stencil testing. | `true`, `false` |
| **StencilReadMask** | Mask for reading stencil buffer values. | `0 → 255` |
| **StencilWriteMask** | Mask for writing stencil buffer values. | `0 → 255` |
| **StencilFailOp** | Action when stencil test fails. | `KEEP`, `ZERO`, `REPLACE`, `INCR_SAT`, `DECR_SAT`, `INVERT`, `INCR`, `DECR` |
| **StencilDepthFailOp** | Action when stencil test passes but depth test fails. | `KEEP`, `ZERO`, `REPLACE`, `INCR_SAT`, `DECR_SAT`, `INVERT`, `INCR`, `DECR` |
| **StencilPassOp** | Action when both stencil and depth tests pass. | `KEEP`, `ZERO`, `REPLACE`, `INCR_SAT`, `DECR_SAT`, `INVERT`, `INCR`, `DECR` |
| **StencilFunc** | Comparison function for front-face stencil test. | `NEVER`, `LESS`, `EQUAL`, `LESS_EQUAL`, `GREATER`, `NOT_EQUAL`, `GREATER_EQUAL`, `ALWAYS` |
| **BackStencilFailOp** | Action for back-facing polygons when stencil test fails. | `KEEP`, `ZERO`, `REPLACE`, `INCR_SAT`, `DECR_SAT`, `INVERT`, `INCR`, `DECR` |
| **BackStencilDepthFailOp** | Action for back-facing polygons when depth test fails. | `KEEP`, `ZERO`, `REPLACE`, `INCR_SAT`, `DECR_SAT`, `INVERT`, `INCR`, `DECR` |
| **BackStencilPassOp** | Action for back-facing polygons when both tests pass. | `KEEP`, `ZERO`, `REPLACE`, `INCR_SAT`, `DECR_SAT`, `INVERT`, `INCR`, `DECR` |
| **BackStencilFunc** | Comparison function for back-face stencil test. | `NEVER`, `LESS`, `EQUAL`, `LESS_EQUAL`, `GREATER`, `NOT_EQUAL`, `GREATER_EQUAL`, `ALWAYS` |
| **StencilRef** | Reference value used in stencil tests. | `0 → 255` |
| **AlphaToCoverageEnable** | Enables alpha-to-coverage for multisampling. | `true`, `false` |
| **BlendEnable** | Enables or disables blending. | `true`, `false` |
| **IndependentBlendEnable** | Allows blending settings to differ between render targets. | `true`, `false` |
| **SrcBlend** | Source factor for RGB blend. | `ZERO`, `ONE`, `SRC_COLOR`, `INV_SRC_COLOR`, `SRC_ALPHA`, `INV_SRC_ALPHA`, `DEST_ALPHA`, `INV_DEST_ALPHA`, `DEST_COLOR`, `INV_DEST_COLOR`, `SRC_ALPHA_SAT`, `BLEND_FACTOR`, `SRC1_COLOR`, `INV_SRC1_COLOR`, `SRC1_ALPHA`, `INV_SRC1_ALPHA` |
| **DstBlend** | Destination factor for RGB blend. | `ZERO`, `ONE`, `SRC_COLOR`, `INV_SRC_COLOR`, `SRC_ALPHA`, `INV_SRC_ALPHA`, `DEST_ALPHA`, `INV_DEST_ALPHA`, `DEST_COLOR`, `INV_DEST_COLOR`, `SRC_ALPHA_SAT`, `BLEND_FACTOR`, `SRC1_COLOR`, `INV_SRC1_COLOR`, `SRC1_ALPHA`, `INV_SRC1_ALPHA` |
| **BlendOp** | Operation applied to RGB blend factors. | `ADD`, `SUBTRACT`, `REV_SUBTRACT`, `MIN`, `MAX` |
| **SrcBlendAlpha** | Source factor for alpha blending. | `ZERO`, `ONE`, `SRC_COLOR`, `INV_SRC_COLOR`, `SRC_ALPHA`, `INV_SRC_ALPHA`, `DEST_ALPHA`, `INV_DEST_ALPHA`, `DEST_COLOR`, `INV_DEST_COLOR`, `SRC_ALPHA_SAT`, `BLEND_FACTOR`, `SRC1_COLOR`, `INV_SRC1_COLOR`, `SRC1_ALPHA`, `INV_SRC1_ALPHA` |
| **DstBlendAlpha** | Destination factor for alpha blending. | `ZERO`, `ONE`, `SRC_COLOR`, `INV_SRC_COLOR`, `SRC_ALPHA`, `INV_SRC_ALPHA`, `DEST_ALPHA`, `INV_DEST_ALPHA`, `DEST_COLOR`, `INV_DEST_COLOR`, `SRC_ALPHA_SAT`, `BLEND_FACTOR`, `SRC1_COLOR`, `INV_SRC1_COLOR`, `SRC1_ALPHA`, `INV_SRC1_ALPHA` |
| **BlendOpAlpha** | Operation applied to alpha blend factors. | `ADD`, `SUBTRACT`, `REV_SUBTRACT`, `MIN`, `MAX` |
| **ColorWriteEnable0** | Channels to write for render target 0. | `false`, `R`, `G`, `B`, `A`, `RG`, `RB`, `RA`, `GB`, `GA`, `BA`, `GBA`, `RBA`, `RGA`, `RGB`, `RGBA` |
| **ColorWriteEnable1** | Channels to write for render target 1. | `false`, `R`, `G`, `B`, `A`, `RG`, `RB`, `RA`, `GB`, `GA`, `BA`, `GBA`, `RBA`, `RGA`, `RGB`, `RGBA` |
| **ColorWriteEnable2** | Channels to write for render target 2. | `false`, `R`, `G`, `B`, `A`, `RG`, `RB`, `RA`, `GB`, `GA`, `BA`, `GBA`, `RBA`, `RGA`, `RGB`, `RGBA` |
| **ColorWriteEnable3** | Channels to write for render target 3. | `false`, `R`, `G`, `B`, `A`, `RG`, `RB`, `RA`, `GB`, `GA`, `BA`, `GBA`, `RBA`, `RGA`, `RGB`, `RGBA` |
| **SrgbWriteEnable0** | Enables sRGB write conversion for render target 0. | `true`, `false` |
| **SrgbWriteEnable1** | Enables sRGB write conversion for render target 1. | `true`, `false` |
| **SrgbWriteEnable2** | Enables sRGB write conversion for render target 2. | `true`, `false` |
| **SrgbWriteEnable3** | Enables sRGB write conversion for render target 3. | `true`, `false` |
| **BlendFactor** | Enables use of a constant blend factor. | `true`, `false` |
| **HighPrecisionBlendEnable** | Enables high-precision blending. | `true`, `false` |
| **AlphaTestEnable** | Enables alpha testing. | `true`, `false` |
| **AlphaTestRef** | Reference value for alpha testing. | `true`, `false` *(likely meant to be a numeric range instead)* |
| **AlphaTestFunc** | Comparison function for alpha testing. | `NEVER`, `LESS`, `EQUAL`, `LESS_EQUAL`, `GREATER`, `NOT_EQUAL`, `GREATER_EQUAL`, `ALWAYS` |
