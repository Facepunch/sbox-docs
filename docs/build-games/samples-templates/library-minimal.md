---
title: Minimal Library Template
sources:
  - game/templates/library.minimal
created: 2026-04-27
updated: 2026-04-27
---

# Minimal Library Template

The **Minimal Library** template (`game/templates/library.minimal/`) is designed for writing code-only extensions or headless utility packages without any associated game assets or startup scenes.

## Quick Working Example

If you are writing a purely mathematical utility library, you would use this template. It also provides a dedicated space for Unit Tests:

```csharp
// UnitTests/LibraryTest.cs
using Sandbox;

[TestClass]
public partial class LibraryTests
{
    [TestMethod]
    public void SceneCreationTest()
    {
        // You can run engine tests headlessly
        var scene = new Scene();
        using ( scene.Push() )
        {
            var go = new GameObject();

            // Verify logic programmatically
            Assert.AreEqual( 1, scene.Directory.GameObjectCount );
        }
    }
}
```

1. **Shared Math/Utility Layers:** If your studio is making three different games in s&box, you can put your shared logic (e.g., custom network serializers, math extensions) into a Library project and include it as a dependency in all three games.
2. **Automated Testing:** Because libraries often don't require user input to function, they are the ideal place to write `[TestMethod]`s. You can run these tests via the s&box Editor's "Test Explorer" window to ensure your logic remains unbroken as the engine updates.

- **Asset Loading in Tests:** Because this template does not launch a full game instance, attempting to test code that relies on `Model.Load()` or querying the active `Game.ActiveScene` may fail or throw NullReferenceExceptions. If a test requires a Scene, you must explicitly construct a temporary one as shown in the example above (`var scene = new Scene(); using ( scene.Push() )`).

## Troubleshooting

:::warning Common Gotchas
- **Tests not appearing in the Test Explorer:**
  Ensure your test class is marked with `[TestClass]` and the methods you want to run are marked with `[TestMethod]`. Note that the class must be `public`.
- **Can't spawn Prefabs in tests:**
  If your library relies on loading `.prefab` files, it needs to be aware of the virtual filesystem mount points. In a pure headless test environment, some addon assets may not be mounted.
:::

## Related Pages
- [Minimal Addon Template](addon-minimal.md)
