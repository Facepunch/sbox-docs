---
title: "Tools (Engine Tooling)"
icon: "🔧"
sources:
  - engine/Tools/
updated: 2026-04-25
created: 2026-04-27
---

# Tools (Engine Tooling)

The `engine/Tools/` directory contains standalone C# applications and scripts used to build and process assets for the engine itself, existing entirely outside of the game execution loop.

## Quick Working Example

As an engine developer, if you modify a native C++ header file and need those functions accessible in C#, you must run the Interop Generator tool located here:

```bash
# Conceptual workflow: Running a tool from this directory
cd engine/Tools/InteropGen
dotnet run --project InteropGen.csproj --source ../../Native/Source2 --output ../../Sandbox.Bind/Generated
```

## Troubleshooting

:::warning
- **"MethodNotFound" after engine compile:** If you added a new C++ function to the native engine but get a `DllNotFoundException` or `MethodNotFound` error in C#, it means you forgot to run `InteropGen` to update the C# bindings, or the C++ macro was incorrect.
- **Bootstrap Failures:** If `Bootstrap.bat` fails, it is almost always due to an error in `SboxBuild` failing to locate the required .NET SDKs or Visual Studio build tools on your machine.
:::

## Tools in this Directory

* [**CodeGen**](codegen.md): The Roslyn-based source generator used to create boilerplate for networking and reflection.
* [**CreateGameCache**](creategamecache.md): A tool for pre-compiling assets into a flat cache file for faster loading in standalone builds.
* [**InteropGen**](interopgen.md): The complex tool that parses native C++ headers and generates the C# P/Invoke bindings (`Sandbox.Bind`).
* [**MenuBuild**](menubuild.md): Scripts for compiling the main menu environment.
* [**SboxBuild**](sboxbuild.md): The primary orchestrator used to build the engine solution and prepare it for distribution.
* [**ShaderCompiler**](shadercompiler.md): Tools that interface with the native Source 2 shader compiler (`fxc` / `dxc`).
* [**ShaderProc**](shaderproc.md): Pre-processor for Shader Graph files (`.shdrgrph`) to convert them into HLSL before compilation.

## Related Pages
- [Pipeline Tools](../../scripting-code-workflows/pipeline-tools.md)
