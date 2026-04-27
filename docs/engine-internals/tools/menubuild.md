---
title: "MenuBuild"
icon: "⚙️"
sources:
  - engine/Tools/MenuBuild/Program.cs
updated: 2026-04-25
created: 2026-04-27
---

# MenuBuild

The `MenuBuild` tool is a specialized pipeline application that compiles the overarching UI and editor projects that make up the s&box main menu and development environment.

## Common Patterns

1. **Output Marshalling (`CopyCompilerOutput`):** 
   When the compiler finishes generating the assemblies for the base and menu projects, it stores them in an internal assembly file system. The `MenuBuild` tool extracts these `.dll` files (`project.AssemblyFileSystem.FindFile`) and physically writes the raw bytes to the correct output path on the host disk. This ensures the engine's runtime can discover and load the compiled menu assemblies when it boots.

## Troubleshooting

:::warning
- **Compile Failures in Menu Code:** If there is a syntax error in `addons/menu` or `addons/base`, this tool will fail during `Project.CompileAsync()`. Because this tool runs *before* the Editor boots, a failure here means the s&box Editor or Main Menu will fail to launch entirely, or will load an outdated cached version of the menu assembly.
:::

## Related Pages
- [Main Menu Addon](../addons/index.md)
- [Tools Addon](../addons/tools.md)
