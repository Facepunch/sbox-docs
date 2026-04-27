---
title: AppSystem
icon: "💻"
sources:
  - engine/Sandbox.AppSystem/AppSystem.cs
  - engine/Sandbox.AppSystem/AppSystemFlags.cs
  - engine/Sandbox.AppSystem/StandaloneAppSystem.cs
created: 2026-04-27
updated: 2026-04-27
---

# AppSystem

The `AppSystem` is a core engine class responsible for initializing the application environment, checking system requirements, and orchestrating the main execution loop for different modes (Editor, Standalone, Dedicated Server, Tool, Qt).

## AppSystem Flags

When an `AppSystem` initializes, it configures itself using `AppSystemFlags`. These flags tell the underlying C++ engine how to behave.

```csharp
public enum AppSystemFlags
{
	None = 0,
	IsConsoleApp = 1 << 0,
	IsGameApp = 1 << 1,
	IsDedicatedServer = 1 << 2,
	IsStandaloneGame = 1 << 3,
	IsEditor = 1 << 4,
	IsUnitTest = 1 << 5,
}
```

## Common Modes

### StandaloneAppSystem
When players run your exported game, this system takes over. It reads the `StandaloneManifest` from your exported game folder, sets the window title, flags itself as `IsGameApp | IsStandaloneGame`, and tells the engine to load your game's package.

### EditorAppSystem
When launching the s&box tools, this system handles initialization. It sets `IsEditor` and handles loading the Qt-based editor interface and the asset browser.

### TestAppSystem
Used internally for unit testing. It brings up a minimal headless version of the engine to run `[TestClass]` methods without opening a window or loading rendering pipelines.

## Troubleshooting

:::warning System Requirements Error
If a player receives an error stating: **"Your CPU needs to support AVX instructions to run this game."**, this is thrown by `AppSystem.TestSystemRequirements()`. The engine physically cannot run on their hardware, as the underlying native binaries are compiled with AVX instructions enabled. They must upgrade their CPU.
:::

## Related Pages
- [Runtime Layout Overview](index.md)
- [Engine Launcher](launcher.md)
