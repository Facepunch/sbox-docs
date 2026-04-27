---
title: "Sandbox.CodeUpgrader/Upgraders"
icon: "🪄"
updated: 2026-04-25
sources:
  - engine/Sandbox.CodeUpgrader/Upgraders/
  - engine/Sandbox.CodeUpgrader/Helpers/Analyzer.cs
  - engine/Sandbox.CodeUpgrader/Helpers/Fixer.cs
created: 2026-04-27
---

# Sandbox.CodeUpgrader/Upgraders

The `Upgraders/` directory contains the concrete migration rules — one pair of classes per deprecated pattern. Each upgrader has two halves:

| Class | Inherits | Job |
|---|---|---|
| `*Analyzer` | [`Analyzer`](sandbox-codeupgrader-helpers.md) | Detect the deprecated syntax and surface a diagnostic. |
| `*Fix` | [`Fixer<TAnalyzer>`](sandbox-codeupgrader-helpers.md) | Provide a code-fix action that rewrites the diagnostic away. |

Both halves are loaded into Roslyn during user-code compilation; together they produce a squiggle in the editor *and* a "Quick Fix" that performs the migration.

## A real upgrader, end to end

Condensed from `engine/Sandbox.CodeUpgrader/Upgraders/AuthorityAttribute.cs` — the rule that migrates legacy `[Authority]` to `[Rpc.Owner]`:

```csharp
namespace Sandbox.CodeUpgrader;

[DiagnosticAnalyzer( LanguageNames.CSharp )]
public partial class AuthorityAttributeAnalyzer : Analyzer
{
    public override DiagnosticDescriptor Rule => Diagnostics.AuthorityAttribute;

    public override void Init( AnalysisContext context )
    {
        context.RegisterSyntaxNodeAction( AnalyzeNode, SyntaxKind.Attribute );
    }

    void AnalyzeNode( SyntaxNodeAnalysisContext context )
    {
        var attr = (AttributeSyntax)context.Node;
        if ( attr.Name.ToString() == "Authority" || attr.Name.ToString() == "Sandbox.Authority" )
            context.ReportDiagnostic( Diagnostic.Create( Rule, attr.GetLocation() ) );
    }

    public override async Task RunTests( IAnalyzerTest tester )
    {
        // Each test feeds the analyzer a snippet with [|...|] markers around
        // the spans it should flag.
        await tester.TestWithMarkup( """
            public class MyClass
            {
                [[|Authority|]]
                public void RunIt(){}
            }
            """ );
    }
}

[ExportCodeFixProvider( LanguageNames.CSharp ), Shared]
public class AuthorityAttributeFix : Fixer<AuthorityAttributeAnalyzer>
{
    public override async Task RegisterCodeFixesAsync( CodeFixContext context )
    {
        var root = await context.Document.GetSyntaxRootAsync( context.CancellationToken );
        var attr = root.FindToken( context.Diagnostics.First().Location.SourceSpan.Start )
            .Parent.AncestorsAndSelf().OfType<AttributeSyntax>().FirstOrDefault();
        if ( attr == null ) return;

        context.RegisterCodeFix(
            CodeAction.Create(
                title: "Upgrade to [Rpc.Owner]",
                createChangedDocument: c => RewriteAsync( context.Document, attr ),
                equivalenceKey: "ChangeAuthorityToRpc",
                priority: CodeActionPriority.High ),
            context.Diagnostics.First() );
    }

    private async Task<Document> RewriteAsync( Document doc, AttributeSyntax oldAttr )
    {
        var newAttr = SyntaxFactory.Attribute( SyntaxFactory.ParseName( "Rpc.Owner" ) );
        var root = await doc.GetSyntaxRootAsync();
        return doc.WithSyntaxRoot( root.ReplaceNode( oldAttr, newAttr ) );
    }
}
```

Key points:

- **`[DiagnosticAnalyzer( LanguageNames.CSharp )]`** is the Roslyn attribute that registers the analyzer. **`[ExportCodeFixProvider( LanguageNames.CSharp ), Shared]`** does the same for the fixer. There is no in-house `[Upgrader]` attribute — both halves use the standard Roslyn registration.
- The `Analyzer` base class hides most of the boilerplate: just override `Rule`, `Init`, and `RunTests`. See [Sandbox.CodeUpgrader/Helpers](sandbox-codeupgrader-helpers.md).
- The `Fixer<TAnalyzer>` base ties a fix-provider to a specific analyzer; `Target.Rule.Id` is wired up for you.
- `Diagnostics.AuthorityAttribute` is one of the project's `DiagnosticDescriptor` constants — collected in a single static class so all upgraders share the same registry.

## What's in the directory

A representative slice (the project's `Upgraders/` is dozens of files):

| File | Migration |
|---|---|
| `AuthorityAttribute.cs` | `[Authority]` → `[Rpc.Owner]` |
| `BroadcastAttribute.cs` | `[Broadcast]` → `[Rpc.Broadcast]` |
| `ConCmdAttribute.cs` | older `[Cmd]` → `[ConCmd]` |
| `ConVarAttribute.cs` | older `[Var]` → `[ConVar]` |
| `HostSyncAttribute.cs` | `[HostSync]` → `[Sync(SyncFlags.FromHost)]` |
| `SyncQuery.cs` | older `[Sync]` → `[Sync(SyncFlags.Query)]` |
| `HotloadUnsupported.cs` | diagnoses patterns hotload can't migrate |
| `GpuBuffer.cs` | older GPU buffer surface |

Each file usually contains the `*Analyzer` + `*Fix` pair. New rules slot in alongside; the convention is one file per deprecated symbol or pattern.

## Adding an upgrader

1. Add a `DiagnosticDescriptor` for your rule to the project's `Diagnostics` static class.
2. Create `MyDeprecatedAnalyzer : Analyzer` decorated with `[DiagnosticAnalyzer(LanguageNames.CSharp)]`. Override `Rule`, `Init`, and `RunTests`.
3. Create `MyDeprecatedFix : Fixer<MyDeprecatedAnalyzer>` decorated with `[ExportCodeFixProvider(LanguageNames.CSharp), Shared]`. Override `RegisterCodeFixesAsync`.
4. Cover the rule with `RunTests` — at minimum the trigger case and the negative case (the fixed form should not re-trigger).

## Maintenance and lifecycle

- **Accumulation.** This directory grows as the public API evolves.
- **Pruning.** Rules that target APIs deprecated years ago are eventually removed once every active project has migrated; pruning is preferable to letting analyzer load-time grow unbounded.
- **Testing.** Every analyzer should ship `RunTests` covering at minimum the trigger case, the negative case, and any edge cases that confused earlier prototypes.

## Common contributor pitfalls

:::warning False positives mangle unrelated code
The most damaging upgrader bug is one that flags a user's *own* property because it shares a name with a deprecated engine member. Use the `SemanticModel` (via `context.SemanticModel`) to verify the declaring type before issuing a diagnostic — string-matching the syntax-node text is brittle.
:::

:::warning Rule-ID collisions
`DiagnosticDescriptor` IDs (`SBxxxx`) must be unique per project. Check `Diagnostics.cs` before allocating a new one.
:::

## Related Pages

- [Sandbox.CodeUpgrader](sandbox-codeupgrader/index.md) — overview of the upgrader pipeline.
- [Sandbox.CodeUpgrader/Helpers](sandbox-codeupgrader-helpers.md) — the `Analyzer` and `Fixer<T>` base classes used here.
