---
title: "Sandbox.Razor"
icon: "🌐"
sources:
  - engine/Sandbox.Razor/Razor.cs
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Razor

The `Sandbox.Razor` namespace contains the engine's customized integration of the Microsoft Razor view engine, which processes `.razor` UI files into executable C# components.

## Compilation Pipeline

The main entry point for the engine is `RazorProcessor.GenerateFromSource`. The process follows these steps:

1.  **Namespace Injection:** If the `.razor` file lacks an explicit `@namespace` directive, `Sandbox.Razor` automatically injects one based on the file's folder structure (e.g., placing a file in `game/ui/menus` will result in the namespace matching the project root plus `.ui.menus`). Invalid directory names (containing spaces or hyphens) are skipped to maintain valid C# identifiers.
2.  **Parsing:** The raw text is converted into a `RazorSourceDocument`.
3.  **Processing:** The `RazorProjectEngine` processes the source document into a `RazorCodeDocument`, evaluating tag helpers and component mappings.
4.  **C# Generation:** Finally, the system extracts the `RazorCSharpDocument` and returns the `GeneratedCode` string, which is then fed into the Roslyn C# compiler.

## Extensibility for Engine Developers

Engine developers modifying `Sandbox.Razor` typically do so to alter how markup is interpreted:

*   **Custom Tag Helpers:** Adding custom tags that map directly to specific C# `Panel` types.
*   **Directive Handling:** Modifying the engine to understand new `@` directives.
*   **CSS Extraction:** Intercepting `<style>` blocks to process and route them to the engine's stylesheet compiler instead of treating them as HTML output.

## Performance

*   **Compile Time vs Runtime:** The Razor processing happens entirely at *compile time* (or hotload time). There is zero Razor parsing happening during the game's actual runtime tick. Therefore, performance concerns regarding `Sandbox.Razor` primarily affect hotload times, not frame rates.
*   **Folder Namespacing:** The automatic namespace calculation (`AddNamespace`) is a fast string operation, but heavily nested folders with invalid C# characters will cause it to repeatedly check and skip segments. 

## Edge Cases & Limitations

:::warning Automatic Namespacing Gotcha
The automatic `@namespace` injection skips folder names that contain spaces, hyphens (`-`), or start with numbers. If your project relies on folder-based namespacing, ensure your UI folder structures strictly follow C# naming conventions, otherwise your Razor components may end up in unexpected namespaces and fail to bind correctly.
:::

## Troubleshooting

*   **"Component not found" errors:** If a C# Component cannot find a corresponding Razor file, check that the `.razor` file compiles. A syntax error inside a `@code` block or malformed HTML will cause `RazorProcessor` to fail silently or generate invalid C#, preventing the panel from appearing.
*   **Namespace Collisions:** If two `.razor` files in different folders accidentally resolve to the same namespace (due to invalid characters being stripped), the compiler will throw duplicate class definition errors.

## Related Pages

*   [UI / Razor Panels (Build Games)](../systems/ui/razor-panels/index.md) - For game developers looking to *use* Razor to build UI.
*   [Sandbox.Compiling](sandbox-compiling.md) - How Razor compilation fits into the broader C# hotload system.
