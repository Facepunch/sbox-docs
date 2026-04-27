---
title: "Sandbox.SolutionGenerator"
icon: "💻"
sources:
  - engine/Sandbox.SolutionGenerator/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.SolutionGenerator

The `Sandbox.SolutionGenerator` is an internal engine tool responsible for dynamically generating `.sln` and `.csproj` files so that external IDEs like Visual Studio, Rider, and VS Code can provide accurate IntelliSense and code navigation for s&box projects.

## Quick Start Example

As an engine developer, you might need to tweak how project references are resolved:

```csharp
using Sandbox.SolutionGenerator;

internal class ProjectGenHook
{
    public void OnProjectGenerated( ProjectConfig project, CsprojFile csproj )
    {
        // Conceptual: Adding a specific MSBuild property dynamically
        csproj.AddPropertyGroup( "LangVersion", "12.0" );
        csproj.AddPropertyGroup( "Nullable", "enable" );
        
        Log.Info( $"Updated CSProj for {project.Ident}" );
    }
}
```

## Common Patterns

1. **Roslyn Analyzers:** The generator automatically links the engine's custom Roslyn analyzers (like `Sandbox.Access` static analysis warnings) into the `.csproj` so the user sees red squigglies in their IDE before they even try to hotload.
2. **Hidden Files:** The generated `.csproj` and `.sln` files are intentionally kept out of source control (via `.gitignore`) because they contain absolute paths to the engine's installation directory on the specific user's computer.

## Visual Diagnostics

The Editor provides a dedicated "Compiler" tab where developers can see a log of the Solution Generator's output. If generation fails, it outputs the exact MSBuild parsing error or file lock exception, allowing engine contributors to quickly diagnose IDE-syncing bugs.

## Troubleshooting

:::warning
- **"The type or namespace name 'Sandbox' could not be found" in IDE:** This happens if the generator failed to run, or if the user moved their s&box installation folder. Deleting the `.sln` and `.csproj` files and letting the editor regenerate them solves 99% of IDE IntelliSense issues.
- **"Reference paths are invalid":** Ensure you do not commit generated `.csproj` files to Git. Another developer pulling your repository will have a different absolute installation path, breaking their project references.
:::

## Sample Validation
Look at tests within `engine/Sandbox.Test/` that mock a complex `.sbproj` hierarchy with circular dependencies to ensure the `SolutionGenerator` does not infinite loop and produces valid XML that MSBuild can parse.

## Related Pages
- [Pipeline Tools](../scripting-code-workflows/pipeline-tools.md)
