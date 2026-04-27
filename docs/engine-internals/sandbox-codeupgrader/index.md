---
title: "Sandbox.CodeUpgrader"
icon: "⬆️"
sources:
  - engine/Sandbox.CodeUpgrader/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.CodeUpgrader

The `Sandbox.CodeUpgrader` is an internal engine Roslyn-based tool that automatically refactors deprecated user C# code to match new API signatures during the compilation pipeline.

## Quick Start Example

As an engine contributor, when you rename an API or change its signature, you must write an upgrader to migrate existing community code automatically:

```csharp
using Sandbox.CodeUpgrader;
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.Diagnostics;

// Real upgraders are a pair: an Analyzer that flags occurrences and a
// Fixer<T> that rewrites them. See engine/Sandbox.CodeUpgrader/Upgraders/SyncQuery.cs
// for a worked example, and engine/Sandbox.CodeUpgrader/Helpers/Analyzer.cs for the base.
[DiagnosticAnalyzer( LanguageNames.CSharp )]
public partial class MyAnalyzer : Analyzer
{
    public override DiagnosticDescriptor Rule => Diagnostics.SyncQuery; // pick or define your own

    public override void Init( AnalysisContext context )
    {
        context.RegisterSyntaxNodeAction( AnalyzeNode, SyntaxKind.InvocationExpression );
    }

    void AnalyzeNode( SyntaxNodeAnalysisContext context )
    {
        // Inspect context.Node and call context.ReportDiagnostic( ... ) when matched.
    }
}
```

## Common Patterns

1. **SyntaxRewriterUpgrader:** The most common pattern. Inherit from `CSharpSyntaxRewriter` to visit specific nodes (like Method Invocations, Property Declarations) and return modified versions.
2. **RegexUpgrader:** For very simple, unambiguous text replacements (like renaming a namespace), the upgrader can fall back to regular expressions via `Sandbox.CodeUpgrader/Helpers/`, though this is generally discouraged as it lacks semantic awareness.

## Diagnostic Feedback

When the Code Upgrader successfully modifies a file, it outputs a log to the s&box Editor Console indicating which file was changed and which upgrader rule fired. This transparency is vital so game developers understand why their local files were modified on disk.

## Troubleshooting

:::warning
- **Infinite Upgrade Loops:** If an upgrader rewrites code into a format that the upgrader itself still considers a match, it will continuously modify the file on every compile. Always ensure your match conditions are specific enough to ignore already-upgraded code.
- **"File changed externally" conflicts:** Because the upgrader modifies the user's `.cs` files on disk, IDEs like Visual Studio will pop up a warning saying the file was changed by another program. This is expected behavior.
:::

## Sample Validation

The upgrader is heavily tested. To see how these AST mutations are validated without destroying code, look at `engine/Sandbox.CodeUpgrader.Test/Tests/`. These unit tests feed string snippets of old code into the upgrader and assert the exact string output, guaranteeing formatting and logic are preserved.

## Subsystems

* [**Helpers**](../sandbox-codeupgrader-helpers.md) — Utility classes for common AST mutations and formatting preservation during the upgrade process.
* [**Upgraders**](../sandbox-codeupgrader-upgraders.md) — The actual rule definitions and `SyntaxRewriterUpgrader` implementations that target and modify specific deprecated patterns.

## Related Pages
- [Sandbox.Compiling](../sandbox-compiling.md)
