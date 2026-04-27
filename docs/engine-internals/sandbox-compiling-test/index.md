---
title: "Sandbox.Compiling.Test"
icon: "🔬"
sources:
  - engine/Sandbox.Compiling.Test/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Compiling.Test

The `Sandbox.Compiling.Test` assembly contains the automated test suite responsible for validating the engine's internal compiler pipeline and dependency resolution graph.

## Quick Working Example

A typical test involves constructing an isolated compilation environment with mock files and verifying the output structure or diagnostics.

```csharp
using Sandbox.Compiling;
using Sandbox.Compiling.Test;
using Microsoft.CodeAnalysis;

[TestClass]
public class CompilerDependencyTests
{
    [TestMethod]
    public void TestAddonDependencyResolution()
    {
        // Conceptual: Setup a mock filesystem where Addon B depends on Addon A
        var projectA = MockProject.Create( "AddonA", "public class ClassA {}" );
        var projectB = MockProject.Create( "AddonB", "public class ClassB : ClassA {}" );
        projectB.AddDependency( projectA );

        var compiler = new CompilerPipeline();
        
        // Compile Addon B and verify that Addon A was implicitly compiled and linked
        var result = compiler.Compile( projectB );

        Assert.IsTrue( result.Success, "Addon B failed to compile." );
        Assert.IsNotNull( result.Assembly.GetType( "ClassB" ), "ClassB missing." );
    }
}
```

## Common Patterns

1. **Mock Projects:** Many tests dynamically generate minimal `.cs` files and project structures in memory or temporary directories to feed into the compiler pipeline, avoiding the need for massive pre-existing test files.
2. **Diagnostic Validation:** Crucial tests intentionally compile invalid C# syntax (like a missing semicolon or incorrect type name) and assert that the `CompilerResult` contains the expected Roslyn diagnostic error codes (e.g., `CS1002`).

## Troubleshooting

:::warning
- **File Locks in Tests:** Since the compiler outputs physical assemblies and metadata files during testing, ensure your tests properly dispose or clear their isolated directories to prevent file lock exceptions on subsequent runs.
:::

## Related Pages
- [Sandbox.Compiling](../sandbox-compiling.md)
