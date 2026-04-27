---
title: "Sandbox.Compiling"
icon: "🔨"
sources:
  - engine/Sandbox.Compiling/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.Compiling

The `Sandbox.Compiling` namespace manages the engine's internal C# compilation, dependency resolution, and the fast-path hotloading pipeline.

## Quick Start Example

If you are writing a custom generator or intercepting the compile pipeline as an engine developer, you interact with `Compiler` tasks:

```csharp
using Sandbox;

// CompilerOutput is the actual result type produced by Compiler.
// See engine/Sandbox.Compiling/CompilerOutput.cs.
internal class CompileLogger
{
    public void OnCompileFinished( CompilerOutput result )
    {
        if ( result.Successful )
        {
            Log.Info( $"Successfully compiled {result.Compiler.AssemblyName}" );
        }
        else
        {
            foreach ( var diag in result.Diagnostics )
            {
                // diag is a Microsoft.CodeAnalysis.Diagnostic
                Log.Error( $"Compile Error: {diag.GetMessage()} at {diag.Location}" );
            }
        }
    }
}
```

## Common Patterns

1. **CodeArchive:** The engine packages compiled code into a `CodeArchive`. This is a compressed format containing the IL and metadata, which is what actually gets distributed to clients when they join a server.
2. **Compiler/Blacklist:** A crucial security step. The compiler enforces the sandbox by preventing user code from referencing unauthorized .NET assemblies (like raw `System.IO` file manipulation, or `System.Reflection.Emit`). If a user tries to add an unauthorized NuGet package to their `.sbproj`, the compiler will reject it here.

## Visual Diagnostics

You can visualize the speed and success of the compiler pipeline directly in the Editor's Console. Whenever you save a `.cs` file, a small gear icon will spin, followed by a green success message with the compilation duration (e.g., `Compiled GameLogic in 142ms`). Errors will halt the hotload and display the exact Roslyn diagnostic trace.

## Troubleshooting

:::warning
- **"Failed to emit assembly":** This error usually indicates a deep failure in the Roslyn pipeline, often caused by a custom source generator crashing or generating invalid syntax trees.
- **"Reference to type 'X' claims it is defined in 'Y', but it could not be found":** Usually caused by a failure in the dependency resolution graph when compiling multiple addons simultaneously. Re-saving the `.sbproj` usually forces a clean rebuild.
:::

## Sample Validation
Look at tests within `engine/Sandbox.Compiling.Test/` to see how the engine validates syntax trees, verifies the blacklist correctly strips banned API usage, and ensures that generated assemblies are structurally valid before they reach the hotloader.

## Related Pages
- [Code Hotloading & Compilation](../scripting-code-workflows/code-hotloading.md)
- [Sandbox.CodeUpgrader](sandbox-codeupgrader/index.md)
