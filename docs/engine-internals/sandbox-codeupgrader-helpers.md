---
title: "Sandbox.CodeUpgrader/Helpers"
icon: "🔧"
sources:
  - engine/Sandbox.CodeUpgrader/Helpers/Analyzer.cs
  - engine/Sandbox.CodeUpgrader/Helpers/Fixer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.CodeUpgrader/Helpers

Two thin wrappers — `Analyzer` and `Fixer` — that all concrete code upgraders inherit from. Their job is to hide the verbose Roslyn `DiagnosticAnalyzer` / `CodeFixProvider` plumbing so that an upgrader can be written by overriding two or three things instead of fifteen.

## `Analyzer`

```csharp
public abstract partial class Analyzer : DiagnosticAnalyzer
{
    public virtual DiagnosticDescriptor Rule { get; }

    public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics
        => ImmutableArray.Create( Rule );

    public sealed override void Initialize( AnalysisContext context )
    {
        context.ConfigureGeneratedCodeAnalysis( GeneratedCodeAnalysisFlags.Analyze );
        context.EnableConcurrentExecution();
        Init( context );
    }

    public virtual void Init( AnalysisContext context ) { }
    public virtual Task RunTests( IAnalyzerTest tester ) => Task.CompletedTask;
}
```

A concrete upgrader's analyzer overrides `Rule` (the `DiagnosticDescriptor`) and `Init(AnalysisContext)` to register its syntax-node action. The base class handles the boilerplate: declaring `SupportedDiagnostics`, opting into generated-code analysis, and enabling concurrent execution.

`RunTests(IAnalyzerTest)` is the unit-test hook — overriding it lets the upgrader declare its own test inputs.

## `Fixer`

```csharp
public abstract partial class Fixer<T> : Fixer where T : Analyzer
{
    public override Type Analyzer => typeof( T );
}
```

Concrete fixers inherit from `Fixer<TAnalyzer>` to bind themselves to a specific analyzer. Two things happen automatically:

- `FixableDiagnosticIds` returns `Target.Rule.Id` — the fixer fixes exactly what the analyzer flagged, no more.
- `GetFixAllProvider()` returns `WellKnownFixAllProviders.BatchFixer` — fix-all-occurrences just works.

`Target` lazily constructs an instance of the analyzer via `Activator.CreateInstance` so the fixer can read its `Rule` without the upgrader needing to wire that explicitly.

## Why this exists

A bare Roslyn analyzer + fix pair is ~80 lines of boilerplate before any actual logic. The wrappers compress that to ~20 lines for the common case. As the source comments put it: *"Wraps DiagnosticAnalyzer to make it less of a pain in the ass"* / *"Wraps CodeFixProvider to make it less of a pain in the ass."*

If you're adding a new code upgrader, you'll inherit from `Analyzer` and `Fixer<TAnalyzer>` here. The actual upgrade implementations live in `engine/Sandbox.CodeUpgrader/Upgraders/`.

## Related

- [Sandbox.CodeUpgrader](sandbox-codeupgrader/index.md)
- [Sandbox.CodeUpgrader/Upgraders](sandbox-codeupgrader-upgraders.md)
