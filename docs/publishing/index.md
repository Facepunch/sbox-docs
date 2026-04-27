---
title: Publishing & Exporting
icon: 📦
updated: 2026-04-25
created: 2026-04-27
---

# Publishing & Exporting

Once you have finished developing your game or addon, you will likely want to share it. s&box allows you to export your projects as standalone applications or publish them as addons/assets to sbox.game.

## Quick Working Example

```json
// Example of a minimal .sbproj file used for publishing
{
  "Title": "My Cool Game",
  "Type": "game",
  "Org": "local",
  "Ident": "my_cool_game",
  "Schema": 1,
  "IncludeSourceFiles": false,
  "Resources": null,
  "PackageReferences": [],
  "EditorReferences": null,
  "IsExecutable": true,
  "ExecutableName": "MyCoolGame",
  "Compiler": {
    "RootNamespace": "Sandbox",
    "DefineConstants": "SANDBOX;ADDON;DEBUG",
    "NoWarn": "1701;1702;1591;",
    "WarningsAsErrors": "",
    "TreatWarningsAsErrors": false,
    "Nullables": false,
    "Whitelist": true,
    "ReturnFalseOnException": false,
    "WaitOnDiagnostic": false
  }
}
```

## Common Patterns

1. **Testing Locally:** Always set your project's Organization (`Org`) to `local` while developing. This ensures your project mounts and plays locally without attempting to sync or conflict with the live asset server.
2. **Publishing Online:** When you are ready to publish, you change your `Org` to your registered developer organization (e.g., `facepunch`) and your `Ident` to a unique lowercase string. Your package is then identifiable worldwide as `org.ident`.
3. **Exporting Standalone:** For Game Projects, you can export a standalone executable that bundles the core engine binaries with your specific game assets. This creates a folder you can zip and distribute anywhere (e.g., itch.io or a dedicated Steam AppID).

## Troubleshooting

:::warning Common Gotchas
- **"Package already exists" Error:** When trying to publish, if you use an Organization and Ident combination that someone else has already published, the upload will fail. Always use a unique Ident.
- **Unintended Source Code Included:** By default, if `IncludeSourceFiles` is true in your `.sbproj`, your raw `.cs` files are uploaded. This is great for open-source addons, but if you wish to keep your game logic closed-source, ensure you untoggle this in the project settings before publishing.
- **Missing Dependencies on Client:** If players cannot connect to your published game, ensure you haven't relied on local dependencies (`local.my_other_addon`) that haven't been uploaded. All dependencies must be publicly published for players to join.
:::

## Export and Publish Guides
- [**Exporting as a Standalone Game**](exporting-standalone.md) - How to export your project to an executable that can be distributed on storefronts like Steam.
- [**Publishing Addons**](publishing-addons.md) - How to create and publish addon content (like maps and models) for existing games.

## Related Topics
- [Build Games](../build-games/index.md) — the developer track index.
- [Monetization](../getting-started/monetization.md)
- [Project Types](../getting-started/project-types/index.md)
