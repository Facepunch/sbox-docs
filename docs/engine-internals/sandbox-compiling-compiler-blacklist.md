---
title: Sandbox.Compiling/Compiler/Blacklist
icon: "🚫"
sources:
  - engine/Sandbox.Compiling/Compiler/Blacklist/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Compiling/Compiler/Blacklist

The `Blacklist` system within the `Compiler` is a syntax-level static analyzer that runs during the Roslyn compilation phase to proactively block known unsafe C# patterns.

## Common Patterns

1.  **Roslyn Diagnostic Analyzers:** Blacklist rules are implemented as standard `DiagnosticAnalyzer` classes. This means the same logic that fails the engine build also powers the red squiggly lines in the developer's IDE (Visual Studio/VS Code).
2.  **Early Rejection:** Failing the build at the syntax level prevents the engine from wasting CPU cycles running code generation or IL verification on an assembly that will ultimately be rejected anyway.

## Related Pages
- [Sandbox.Compiling/Compiler](sandbox-compiling-compiler.md)
