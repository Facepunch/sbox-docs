---
title: "Properties"
icon: "🔧"
created: 2025-08-20
updated: 2025-08-20
---

# Properties

There are a few basic properties that each Shader Graph has that affect the way the resulting Shader is drawn and how it can be used.

![](./images/properties.png)

# Blend Mode

Determines whether or not the Shader should support Opacity, and how it should be handled.

![Blend mode comparison](./images/blend-mode-comparison.mp4)

* Opaque - Always full opacity, no support for transparency.
* Masked - No transparency support, but will use dithering with anything in-between 0 and 1.
* Translucent - Full transparency support, allows you to see-through any opacities between 0 and 1.

# Shading Model

Determines whether or not the Shader should be affected by lighting.

![Shading model lighting effects](./images/shading-model-lighting-effects.mp4)

* Lit - Light will affect the material and can support Emissions, Normals, Roughness, Metalness, and Ambient Occlusion.
* Unlit - The material will output the raw colour to the screen regardless of the lighting in the Scene.

# Domain

Determines how/where the shader should be used.

![Shader domain options](./images/shader-domain-options.mp4)

* Surface - Used for any materials that will be used on 3D Objects placed within your scene. Can have a set Shading Model.
* Post Process - Used for post-processing materials that will be applied to the screen. Uses Screen Coordinates instead of Texture Coordinates.
