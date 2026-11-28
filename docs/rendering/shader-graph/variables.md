---
title: "Variables"
icon: "💾"
created: 2025-08-20
updated: 2026-04-10
---

# Variables

Variables are useful for making your Graph easier to follow and understand, while also giving the option to expose certain values to the Material Editor and Code.

# Constants

Any Constant nodes created within a Shader will be compiled as constant values in shader code. These values cannot be changed

![](./images/constants.png)

# Variables

Any Constant nodes that are given a name are automatically exposed in the Material Editor. these values can be customized when creating a Material from the Shader.

![Giving a Float constant the name "Intensity"](./images/giving-a-float-constant-the-name-intensity.png)

![How the Variable appears in the Material Editor](./images/how-the-variable-appears-in-the-material-editor.png)

If a Constant has "Is Attribute" set to true, then the value will not be exposed in the Material Editor and will instead be accessible via `RenderAttributes` in code.

:::warning
If you need to access a variable in code without enabling "Is Attribute" (e.g. via `Material.Set()`), be aware that the Name you assign in ShaderGraph may be automatically modified during shader compilation to follow internal naming conventions.

This means the name you see in ShaderGraph is not always the exact name expected by the [Material API](https://sbox.game/api/Sandbox.Material), which can cause invocations like `Material.Set()` to fail silently.

For example, a float named "Intensity" may be compiled as "g_fIntensity", and must be accessed like this:
`Material.Set("g_fIntensity", newValue)`

To find the exact name needed, open your generated `.shader` file and inspect the variable definitions within.

:::
