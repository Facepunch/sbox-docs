---
title: "InteropGen"
icon: "⚙️"
sources:
  - engine/Tools/InteropGen/
created: 2026-04-27
updated: 2026-04-27
---

# InteropGen

This section covers the internal implementation of the `InteropGen` pipeline tool within the s&box engine.

## Architecture

This folder contains the `InteropGen` C# tool. It is a standalone application responsible for generating the C# to C++ interop bindings (`Sandbox.Bind`) that bridge the managed .NET Core world with the unmanaged Source 2 native engine.

The tool parses custom `.def` (Definition) files that describe native classes, methods, and properties, and automatically writes the corresponding boilerplate code for both sides.

## Common Patterns

1. **Definition Parsing:** `InteropGen` reads a `manifest.def` file inside the target directory (usually `engine/Definitions/`), which lists individual component definition files.
2. **Parallel Processing:** It uses `Task.Run` to process each definition file concurrently, significantly speeding up the generation phase during engine compiles.
3. **Multi-Language Output:** For each definition file, `InteropGen` creates:
    - The managed C# wrapper (`ManagerWriter`) that exposes native methods to `Sandbox.Engine`.
    - The native C++ header file (`NativeHeaderWriter`) declaring the export structures.
    - The native C++ source file (`NativeWriter`) implementing the glue functions that the C# code calls via P/Invoke.

*Engine contributors modifying native engine systems must use this tool (often executed automatically by `SboxBuild` or MSBuild tasks) to ensure their C++ changes are safely accessible from C#.*
