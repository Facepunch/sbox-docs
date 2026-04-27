---
title: "Exporting as a Standalone Game"
icon: "🧍"
created: 2025-10-10
updated: 2026-04-27
sources:
  - engine/Sandbox.Tools/Utility/Standalone/StandaloneExporter.cs
  - engine/Sandbox.Tools/Utility/ExportConfig.cs
---

# Exporting as a Standalone Game

Export your game as a standalone executable to distribute outside of the s&box platform — on Steam or any storefront that also supports Steam.

:::warning
Standalone exporting is currently in preview. Distribution requires approval from Valve, which we're working with them to open up. **Don't distribute exported builds publicly without a license from us — contact `garry@facepunch.com` to discuss.**
:::

## Why Export Standalone

* **Steam and beyond** — publish to any storefront, as long as your game is also on Steam.
* **No engine royalties** — the engine is royalty-free, with no revenue share back to Facepunch.
* **Open source engine** — full access to the engine source code.
* **Full .NET access** — no code whitelist (see [Disabling the Whitelist](#disabling-the-whitelist) below).
* **PC only** — no console or mobile exports right now; that may change in the future.

## Quick Working Example

While exporting is mostly an editor process, you can write code that only executes in your standalone build using the `STANDALONE` preprocessor directive.

```csharp
using Sandbox;

public sealed class StandaloneFeature : Component
{
	protected override void OnStart()
	{
#if STANDALONE
		Log.Info( "Running in an exported standalone game!" );
		// Initialize Steamworks, rich presence, or standalone-only APIs here
#else
		Log.Info( "Running within the s&box platform." );
#endif
	}
}
```

### Using the Export Wizard

Exporting your game is handled entirely through the Export Wizard in the Editor.

1. In the Editor, click on the **Project** menu at the top.
2. Click **Export…**
3. In the wizard, provide the configuration:
   - **Target Directory:** Where to put the exported files.
   - **Executable Name:** The name of the final `.exe`.
   - **Executable Icon:** The icon for your game executable.
   - **Startup Image:** A splash screen to show during load.
   - **Steam App ID:** If you are exporting to Steam, insert your App ID here.
4. Click **Next** and wait for the project export process to complete.
5. Your project's executable will be in the folder you selected. Double click the `.exe` to play it!

![](../images/how-to-export-your-game.png)

## Configuration / Restrictions

The following restrictions apply to all games that are exported from s&box:

* **Steam Distribution:** Your game must be put on Steam (it can be on other storefronts too, but Steam is a hard requirement).
* **Facepunch Account:** You need an active Facepunch developer account to access export tools.
* **Licensing:** You must agree to the standalone version license agreement before distribution.

## Disabling the Whitelist

Platform games enforce an [API whitelist](../code/code-basics/api-whitelist.md) that blocks potentially dangerous .NET APIs. Standalone games can disable this restriction entirely, giving you full access to the .NET runtime.

To disable it, open your project settings and turn off the whitelist option.

:::danger
Once the whitelist is disabled, your game **cannot be published to the s&box platform** in that state. The whitelist exists so that platform users are protected — disabling it is only suitable for builds you intend to distribute exclusively as standalone executables.
:::

## Troubleshooting

:::warning "Do not distribute yet!"
You can export your game locally to test it, but don't distribute exported builds publicly just yet — the standalone license agreement is still being finalized. Contact `garry@facepunch.com` if you need a license now.
:::

:::danger "Build Failed: Unauthorized"
If the export wizard fails immediately, ensure you are logged in to the Editor with a valid developer account.
:::

## Related Pages
- [Publishing Addons](publishing-addons.md)
- [Game Project](../getting-started/project-types/game-project.md)
- [API Whitelist](../code/code-basics/api-whitelist.md)
- [Standalone Builds (Runtime API)](../systems/platform/standalone.md)