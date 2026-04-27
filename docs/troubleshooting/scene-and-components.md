---
title: Scene & Components
icon: "🐛"
created: 2026-04-25
updated: 2026-04-25
---

# Scene & Components

Lifecycle bugs, missing components, prefabs spawning at the wrong place, transforms behaving oddly.

## Component lifecycle

### "My component isn't showing up in the Add Component menu"

The C# project failed to compile. The Editor only loads components from a clean build.

- **Check:** the Editor console for red compile errors. Hotloading pauses entirely until they're fixed.
- **Check:** your file is saved to disk. The hotloader watches disk, not the IDE buffer.
- **Check:** your class is `public` and inherits from `Component`.

**Deep dive:** [Create a Component](../how-to/create-a-component.md), [Hotloading](../code/code-basics/hotloading.md).

### "NullReferenceException in OnAwake"

`OnAwake` runs immediately when the component is created — sibling components on the same GameObject may not be initialized yet.

- **Fix:** move dependency-on-other-components logic to `OnStart`, which runs after the whole GameObject is ready.
- **Check:** are you `[Property]`-bound to a GameObject in the Inspector that you forgot to assign?

**Deep dive:** [Component Lifecycle](../scene/components/component-lifecycle.md).

### "My component code isn't running at all"

- **Check:** both the Component *and* its parent GameObject must be **Enabled**. Disabled either one and `OnUpdate` stops.
- **Check:** the GameObject's parent isn't disabled higher up the hierarchy — disabling cascades.

**Deep dive:** [Components Overview](../scene/components/index.md).

### "My update method runs at unpredictable times relative to other components"

`OnUpdate` order between components is **not deterministic**. Don't rely on Component A's `OnUpdate` running before Component B's.

- **Fix:** use `[ExecuteInEditMode]` ordering attributes, an explicit `OnLateUpdate`, or chain via events.
- **Less common:** use `OnFixedUpdate` only when the work needs to align with physics ticks — note it runs at a fixed rate (e.g., 50Hz), not per-frame.

**Deep dive:** [Execution Order](../scene/components/execution-order.md).

### "Heavy work in OnUpdate is killing my framerate"

`OnUpdate` runs every frame on every enabled component. Don't `Components.GetAll<T>()` or do file I/O in there.

- **Fix:** cache references in `OnStart`. Re-query only when the scene actually changes.
- **Fix:** consider an event-driven approach — `OnTriggerEnter`, RPC callbacks, etc.

**Deep dive:** [Component Lifecycle](../scene/components/component-lifecycle.md), [Component Queries](../how-to/component-queries.md).

## Prefab and clone

### "My prefab spawns at 0,0,0 instead of where I want it"

`MyPrefab.Clone()` with no arguments spawns at world origin.

- **Fix:** `MyPrefab.Clone( WorldTransform )` or `MyPrefab.Clone( position )`.
- **Fix:** alternatively, set `clone.WorldPosition = ...` immediately after cloning.

**Deep dive:** [Spawn a Prefab](../how-to/spawn-prefab.md), [Cloning](../scene/cloning.md).

### "My prefab is null when trying to spawn"

The `[Property]` field referencing your prefab is unassigned in the Inspector.

- **Check:** drag your prefab onto the property in the Inspector. Calling `.Clone()` on null throws.
- **Defensive code:** `if ( PrefabToSpawn is null ) return;`

### "Other players can't see the spawned object"

You forgot to network-spawn it. Cloning is local-only by default.

- **Fix:** call `clone.NetworkSpawn()` after cloning.
- **Check:** the prefab itself is configured as a Network Object.

**See also:** [Networking → Synced var not syncing](./networking.md).

### "I can't delete or move children of a prefab in the scene"

By design — a prefab instance is linked to its master template. The hierarchy is locked.

- **Fix:** double-click the prefab to open and edit the template directly, or unlink the instance.

**Deep dive:** [Prefabs](../scene/prefabs/index.md).

### "Clone is instantly destroyed"

Cloning from inside `OnDestroy` (or another tear-down callback) on an object being destroyed will often produce a clone that also dies.

- **Fix:** clone before destroying, not during. Or clone from a different, still-alive component.

## GameObject queries and lookup

### "Scene.GetAll returns empty"

`Scene.GetAll<T>()` only returns **active and initialized** components on **enabled** GameObjects.

- **Check:** target GameObject and Component are both enabled.
- **Check:** if the component was just created this frame, it may not be initialized yet — try next frame, or query in `OnStart`.

**Deep dive:** [Scenes](../scene/scenes/index.md).

### "FindByName returns nothing or weird results"

`Scene.Directory.FindByName` returns an `IEnumerable<GameObject>` — multiple objects can share the name.

- **Fix:** `.FirstOrDefault()` if you only want one, or iterate.
- **Fix:** prefer a `[Property] GameObject Target` field assigned in the Inspector instead of name lookup.

### "Component.Resolve() returns null"

The component reference is stale — the target was destroyed, the scene was unloaded, or it's a network reference for an object not yet spawned on this client.

- **Check:** `Component.IsValid` before use.

**Deep dive:** [Component Reference](../scene/components/component-reference.md).

### "NullReferenceException when accessing Components.Get&lt;T&gt;()"

`Get<T>()` returns null if the component isn't on that GameObject.

- **Fix:** `if ( !go.Components.TryGet( out MyComp comp ) ) return;` or use `GetOrCreate<T>`.
- **Defensive code:** always null-check before chaining property access.

**Deep dive:** [Components Reference](../scene/components/reference/index.md).

### "Querying components every frame is slow"

`GetInDescendants<T>()` walks the hierarchy. On big trees, this kills perf.

- **Fix:** cache in `OnStart`. Invalidate only when children change.

**Deep dive:** [Component Queries](../how-to/component-queries.md).

## Tags and transforms

### "Tag I removed from the child is still there"

Tags are inherited. The tag is on the parent, and children see it.

- **Fix:** remove the tag from the parent (or whichever ancestor actually has it).
- **Note:** all tags are stored lowercase. `Has("Player")` and `Has("player")` are identical.

**Deep dive:** [Tags](../scene/tags.md).

### "GameObject.Transform.Position is marked obsolete"

Older tutorials are out of date. Access transform values directly on the GameObject:

- `GameObject.WorldPosition`, `GameObject.LocalPosition`
- `GameObject.WorldRotation`, `GameObject.LocalRotation`

**Deep dive:** [Transform](../scene/transform.md).

### "Transform cannot be assigned to a child of itself"

The engine blocks circular hierarchies. You're trying to parent A under B where B is already under A.

- **Fix:** unparent first, or pick a non-cyclical target.

### "I can't add or remove from GameObject.Children directly"

The `Children` collection is read-only in the API sense — modifying parenting goes through `SetParent`.

- **Fix:** `child.SetParent( newParent )` to move, `child.Destroy()` to remove.

### "My Rigidbody ignores Transform changes"

Setting `WorldPosition` on a Rigidbody is a *teleport*. Physics will allow it but it skips collision and may produce instability.

- **Fix:** apply forces, use `Rigidbody.SmoothMove`, or accept teleport semantics for one-shot moves.

**See also:** [Physics → Rigidbody not moving smoothly](./physics.md).

## Scene loading

### "Scene.LoadFromFile: Couldn't find scene"

Path is wrong. Paths are relative to the project root, including extension.

- **Fix:** use a `[Property] SceneFile` instead of a hand-typed string — the Inspector drag-drop guarantees a valid path.
- **Check:** spelling, casing, and `.scene` extension.

**Deep dive:** [Change Scenes](../how-to/change-scenes.md).

### "Multiplayer clients desync or crash on scene change"

`Game.ActiveScene.Load()` only loads the scene locally. On a host, that desyncs all clients.

- **Fix:** the host calls `Game.ChangeScene( options )`. Clients receive the change automatically.

### "Scene transition hangs forever"

`OnLoad` returns a `Task` and the engine waits for it. An infinite loop or hung HTTP request stalls the load forever.

- **Fix:** add a `CancellationToken` and a timeout to async operations in `OnLoad`.
- **Avoid:** spawning gameplay objects in `OnLoad`. Reserve `OnLoad` for asset preload; do gameplay in `OnStart`.

**Deep dive:** [Async Component Loading](../how-to/async-loading.md).

## "Missing Component" errors

### "Component is Missing — red warning in the Inspector"

The C# class for that component no longer exists or failed to compile.

- **Fix the class:** restore the class name, or write a `[JsonUpgrader]` to migrate old serialized data.
- **Remove:** if the component is genuinely gone, right-click → Remove on the missing entry.

**Deep dive:** [Component Versioning](../scene/components/component-versioning.md).

### "My JsonUpgrader doesn't run"

Your `ComponentVersion` doesn't match the version the upgrader targets.

- **Check:** `static int ComponentVersion => N;` and `[JsonUpgrader( N )]` reference the same number.
- **Check:** the upgrader method is `static` and takes one `JsonObject` argument.

## Related Pages

- [Networking](./networking.md)
- [Physics](./physics.md)
- [Hotload & Compile](./hotload-and-compile.md)
- [UI / Razor](./ui-razor.md)
