---
title: Sandbox.Compiling/Compiler
icon: "🔨"
sources:
  - engine/Sandbox.Compiling/Compiler/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Compiling/Compiler

The `Compiler` module wraps the Microsoft Roslyn compiler infrastructure to turn community-provided C# source files into executable assemblies tailored for the s&box environment.

## Common Patterns

1.  **Roslyn Hosting:** Translates engine-specific syntax trees and semantic models using standard Microsoft `Microsoft.CodeAnalysis` APIs, keeping the engine closely aligned with the latest C# standards.
2.  **Incremental Compilation:** Caches syntax trees for files that haven't changed, significantly reducing the latency between hitting "Save" in an IDE and the code running in the game.
3.  **Blacklist Enforcement:** At the syntax level, the compiler immediately rejects attempts to use forbidden keywords or types, failing the build before it ever reaches the `Sandbox.Access` IL verification stage.

## Related Pages
- [Sandbox.Compiling](sandbox-compiling.md)
