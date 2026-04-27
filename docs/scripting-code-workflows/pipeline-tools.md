---
title: Pipeline Tools
icon: "🔧"
sources:
  - engine/Tools
created: 2026-04-27
updated: 2026-04-27
---

# Pipeline Tools

Pipeline tools are standalone utilities and code generators used during the engine's build process.

## Quick Start Example

```csharp
// Example of how InteropGen generates C# bindings for native C++ calls
// This is automatically generated code!

using System.Runtime.InteropServices;

namespace Sandbox.Native
{
    internal static class NativeMethods
    {
        [DllImport("engine", CallingConvention = CallingConvention.Cdecl)]
        public static extern void engine_initialize();
    }
}
```

## Troubleshooting

:::warning Build Failures
If you are contributing to the engine and pull latest changes, you might get immediate build errors complaining about mismatched signatures or missing native methods. This usually means `InteropGen` needs to be run, or your local `engine/Tools/` binaries are out of date and need to be recompiled before they can process the main engine code. Always run `Bootstrap.bat` to ensure the toolchain is fresh.
:::
