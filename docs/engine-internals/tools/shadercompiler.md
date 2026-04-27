---
title: "Shader Compiler Wrapper"
icon: "⚙️"
sources:
  - engine/Tools/ShaderCompiler/Program.cs
created: 2026-04-27
updated: 2026-04-27
---

# Shader Compiler Wrapper

The `ShaderCompiler` tool is a standalone command-line application that acts as a wrapper around the native Source 2 shader compilation process.

## Quick Start Example

When running manually from the command line, you can pass several flags to control the compilation behavior:

```bash
# Compile shaders, forcing a full recompile instead of using cached results
dotnet run --project engine/Tools/ShaderCompiler/ShaderCompiler.csproj -- -f "path/to/shader.vfx"

# Compile shaders on a single thread (useful for debugging compilation crashes)
dotnet run --project engine/Tools/ShaderCompiler/ShaderCompiler.csproj -- -s "path/to/shader.vfx"

# Quiet mode: suppress console output
dotnet run --project engine/Tools/ShaderCompiler/ShaderCompiler.csproj -- -q "path/to/shader.vfx"
```

## Common Patterns

1. **Batching:** The tool is designed to accept multiple file paths in a single invocation. The engine build pipeline typically gathers all modified shader files and passes them to this tool at once, allowing it to compile them concurrently.
2. **Failure Tracking:** The program tracks failed compilation processes (`List<ProcessList> failedList`). If a shader fails due to a syntax error in the HLSL/GLSL code, the wrapper catches this state to report back to the main `SboxBuild` pipeline so the build halts appropriately.

## Troubleshooting

:::warning
- **Silent Failures in Quiet Mode:** If you use the `-q` flag and a shader fails to compile, the tool will exit without printing the underlying HLSL syntax error. If you are experiencing black materials or missing shaders, always run the compiler without `-q` to see the actual error output.
:::

## Related Pages
- [ShaderProc Pre-processor](shaderproc.md)
