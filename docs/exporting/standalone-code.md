---
title: "Standalone Code"
icon: "🔓"
created: 2026-04-26
updated: 2026-04-26
---

# Standalone Code

When your game is exported as a standalone build, it runs outside of the s&box platform. This gives you access to APIs and capabilities that aren't available in platform games.

## STANDALONE Constant

Exported games are compiled with a `STANDALONE` constant. Use this to branch logic between platform and standalone builds.

```csharp
#if STANDALONE
// code that only runs in an exported game
#endif
```

## Disabling the Whitelist

Platform games enforce an [API whitelist](/code/code-basics/api-whitelist.md) that blocks potentially dangerous .NET APIs. Standalone games can disable this restriction entirely.

To disable the whitelist, open your project settings and turn off the whitelist option. This gives you full access to the .NET runtime - but your game won't be publishable to the s&box platform while the whitelist is disabled.
