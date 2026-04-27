---
title: "FGD & GameData Architecture"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/GameData/GameData.cs
  - engine/Sandbox.Tools/GameData/EntityParser.cs
created: 2026-04-27
updated: 2026-04-27
---

# FGD & GameData Architecture

The `Sandbox.Tools.GameData` subsystem bridges modern C# `Component` metadata with the legacy, native C++ structure required by the Hammer Map Editor. It dynamically generates Forge Game Data (FGD) classes in memory based on the managed codebase.

## The Process

1. **Reflection Scan:** When the game assembly compiles, `GameData.Assembly.cs` triggers a scan.
2. **Metadata Construction:** `EntityParser` reads C# attributes (e.g., `[Icon]`, `[Category]`, `[Description]`) to flesh out the `MapClass`.
3. **Native Marshalling:** `MapClass.Native.cs` packages the managed `MapClass` into a C++ `CMapClass` pointer and registers it with the native Hammer environment.
4. **Hammer Consumption:** The Hammer editor UI updates its Entity Tool list and Object Properties panel dynamically based on the newly generated data.

## Troubleshooting

:::warning
- **Missing Properties in Hammer:** If a `[Property]` appears in the standard Inspector but is missing from Hammer, verify the type is supported by the native parser. Unsupported types (like nested object collections) will silently fail to generate a `MapClassVariable`.
- **Duplicate Entities on Hotload:** A bug where a component appears twice in Hammer's entity list is typically caused by a failure in the `GameData` hotload flush cycle, meaning the old `CMapClass` pointer was not successfully destroyed before the new one was registered.
:::

## Related Pages
- [MapEditor (Hammer Integration)](mapeditor.md)
