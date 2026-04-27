---
title: "ShaderProc Pre-processor"
icon: "⚙️"
sources:
  - engine/Tools/ShaderProc/Program.cs
  - engine/Tools/ShaderProc/CppPacker.cs
created: 2026-04-27
updated: 2026-04-27
---

# ShaderProc Pre-processor

The `ShaderProc` tool is a pre-processing pipeline application that parses shader source files and generates packed Interop C++ header files. It runs as part of the engine build pipeline.

## Troubleshooting

:::warning
- **"Definition files was not found!"**:
  The tool expects a strictly defined folder structure. If the `Definitions/shaders.def` file is missing from the directory you pass to the tool, it will throw an exception and halt the pipeline.
:::

## Related Pages
- [Shader Compiler Wrapper](shadercompiler.md)
