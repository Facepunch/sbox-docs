---
title: "SboxBuild Orchestrator"
icon: "⚙️"
sources:
  - engine/Tools/SboxBuild/Program.cs
  - engine/Tools/SboxBuild/Pipeline.cs
created: 2026-04-27
updated: 2026-04-27
---

# SboxBuild Orchestrator

The `SboxBuild` tool is the central command-line orchestrator that builds, formats, and deploys the s&box engine and its managed/native components. It is meant to be run from the root of the s&box project repository.

## Quick Start Example

As an engine contributor, you can invoke `sboxbuild` from the terminal (often handled via `Bootstrap.bat` or CI scripts). For example, to build the engine locally:

```bash
# Build both managed and native code using the Developer configuration
sboxbuild build --config Developer

# Skip the native C++ build if you are only working on C#
sboxbuild build --skip-native
```

## Common Patterns

1. **Pipeline & Steps:** 
   The `Pipeline` class maintains a list of `Step` objects (e.g., `CompileNativeStep`, `CompileManagedStep`). The pipeline executes these sequentially, tracking execution times via `Stopwatch` and determining whether to halt on failure.
2. **Integration Logging:**
   `SboxBuild` includes robust integrations for Continuous Integration (CI). It has specific logging mechanisms for `GitHub.cs` (to write GitHub Action step summaries and limit output to `MAX_GITHUB_FIELD_LENGTH`) and `Slack.cs` to ping developers when a build fails or succeeds.

## Troubleshooting

:::warning Common Gotchas
- **"Command not found: sboxbuild"**:
  If running manually, ensure you have compiled the `SboxBuild.csproj` first, or run it via `dotnet run --project engine/Tools/SboxBuild/SboxBuild.csproj -- build`.
- **"Native compilation failed"**:
  Usually indicates a syntax error in the Source 2 C++ codebase. The tool will halt the pipeline and will not proceed to compile managed code. Check the console output for the exact MSVC error.
:::

## Related Pages
- [Engine Launcher](../launcher.md)
- [Shader Compiler Wrapper](shadercompiler.md)
