---
title: "A Tour of the Editor"
icon: "🗺️"
created: 2026-04-25
updated: 2026-04-25
---

# A Tour of the Editor

You've installed s&box and clicked the launch button — and now you're staring at a complex window with several panels you've never seen. This page is the orientation. Five panels, two toolbars, four menus you'll actually use. Everything else can wait.

If you came straight from [First Steps](first-steps/index.md), you already have the core concepts (scenes, GameObjects, components, hotload). This page is the *spatial* version of that — where each of those concepts lives on screen.

## What you see at first launch

When `sbox-dev.exe` opens with no project selected, you land on the **Welcome screen**:

- **New Project** — opens a wizard for creating a fresh project ([Game Project](project-types/game-project.md), [Addon Project](project-types/addon-project.md), or one of the engine-internal types).
- **Open Project** — browse for an existing `.sbproj` file.
- A list of recent projects, plus links to the testbed, samples, and Discord.

Once you create or open a project, the Welcome screen disappears and you see the main editor layout described below.

> **Pause and predict.** What's the difference between launching `sbox-dev.exe` and launching `sbox.exe`?
>
> <details><summary>Answer</summary>
>
> `sbox-dev.exe` boots the engine in **editor mode** — Qt UI, Inspector, Mapping tool, hotload, all the development chrome. `sbox.exe` boots the engine in **player mode** — no editor, no overlays, just the runtime your shipped game uses. Same install, two launchers; you'll use `sbox-dev.exe` 99% of the time during development.
>
> </details>

## The five panels you'll use most

The default layout has five docked panels (plus a slim top toolbar). They're all detachable and rearrangeable, but the defaults are good:

| Where | Panel | Purpose |
|---|---|---|
| Top edge of the window | Top toolbar | Play, Stop, Pause; scene controls. |
| Left | Scene Tree | Hierarchy of GameObjects in the open scene. |
| Centre | Viewport | The 3D scene itself. |
| Right | Inspector | Components and properties of the selected GameObject. |
| Bottom (left half) | Asset Browser | Files in your project. |
| Bottom (right half) | Console | Compile output, logs, errors. |

### Scene Tree (left)

Every GameObject in the currently-open scene, as a hierarchy. Click one to select it; drag one onto another to re-parent it; right-click for `Create → Empty GameObject`, `Create → 3D Object`, etc.

What goes here: every "thing" in your level — the player, lights, the world geometry, particle effects, UI canvases.

### Inspector (right)

Shows the selected GameObject's components and properties. This is where the `[Property]` attributes you write in C# turn into editable fields. Drag values, paste references, expand collapsed groups.

If nothing's selected, the Inspector is blank. If multiple objects are selected, the Inspector shows shared properties — editing them edits all selected.

**Add Component** at the bottom is how you attach behaviour to a GameObject. Search by name; results include both built-in components (`ModelRenderer`, `Rigidbody`, `PointLight`...) and your own once they compile.

### Viewport (center)

The 3D scene. This is what your camera sees in play mode and what you'd be looking at if your game were running right now.

| Action | How |
|---|---|
| Look around | Right-click + drag |
| Move camera (while right-mouse held) | `WASD` + mouse |
| Frame the selection | `F` |
| Cycle viewmode (lit / wireframe / etc.) | `Ctrl + Space` |
| Toggle gizmos | `Shift + G` |

The viewport is also where you click to select objects and where the Mapping tool draws its handles.

### Asset Browser (bottom-left)

The files in your project. `.scene`, `.prefab`, `.cs`, `.vmdl`, `.vmat`, `.sound`, etc. Right-click to **Create → C# Component**, **Create → Scene**, etc. Double-click an asset to open it.

The asset browser is also how you assign assets to properties: drag a `.vmdl` from here into a `ModelRenderer`'s **Model** field in the Inspector and it binds.

### Console (bottom-right or merged with Asset Browser)

Compile output, `Log.Info` calls, warnings, errors. **Watch this.** When hotload pauses (it pauses entirely on any compile error), the red text here is the only signal something broke.

A console error blocks every subsequent code change from taking effect until you fix it.

## The top toolbar

Slim row at the top of the window. The only buttons newcomers need to know:

| Button | What it does |
|---|---|
| ▶ **Play** (`F5`) | Enters play mode — your scene starts ticking, components run their lifecycle, the camera switches to your `CameraComponent`. |
| ■ **Stop** (`F5` again) | Exits play mode. The scene snaps back to its pre-play state — runtime changes are discarded. |
| ⏸ **Pause** | Freezes the simulation while keeping inspector edits live. |

You'll spend a lot of your day on the Play/Stop cycle. Hotload edits while playing affect the live scene; they don't persist when you stop.

## The menus

Most menus are reference until you need them. The ones worth knowing on day one:

- **File** → **New Scene**, **Save Scene** (`Ctrl + S`), **Save Scene As**, **Open Scene**.
- **Edit** → **Undo** (`Ctrl + Z`), **Redo** (`Ctrl + Y`), **Editor Preferences** (where you configure your code editor — VS Code, Visual Studio, Rider).
- **View** → toggles for the Asset Browser, Console, and other docks if you accidentally close one.
- **Project** → settings for the active project, including `Generate Solution` (regenerate the `.csproj`/`.sln` files your IDE needs).

If your IDE complains it can't find the `Sandbox` namespace, the answer is almost always **Project → Generate Solution**.

## Where common things live

Quick lookup table for "I want to do X, where does that live?":

| Goal | Where |
|---|---|
| Create a GameObject | Scene Tree → right-click → Create → Empty GameObject |
| Attach a component | Select GameObject → Inspector → Add Component |
| Open a scene | Asset Browser → double-click a `.scene` file |
| Save your scene | `Ctrl + S` (or File → Save Scene) |
| Run the game | `F5` (or the Play button) |
| Open the Mapping tool | Viewport → toolbar at the top of the viewport ([Scene Mapping](../scene/scene-mapping/index.md)) |
| Configure your IDE | Edit → Editor Preferences → Code |
| See compile errors | Console panel (bottom) |
| Generate `.sln` for your IDE | Right-click `.sbproj` in Asset Browser → Generate Solution |
| Reset the editor layout | View → Reset Layout |

## Navigating the viewport, in detail

The viewport camera doesn't behave like a video-game player. It's an editor camera — designed for fast scene authoring, not realistic movement:

- **Right-click + drag** rotates the camera. WASD only moves the camera *while right-click is held*.
- **F** with something selected frames the camera on it. The fastest way to get back to your work after losing yourself.
- **Scroll wheel** dollies forward/back.
- **Middle-click + drag** pans.

You're moving the editor camera, not a player. When you press **Play**, the runtime camera (`CameraComponent` in your scene) takes over.

> **Pause and predict.** You're in play mode. You move your character with WASD. You press Stop. The character is now in a different position than where you started. After you press Play again, where is the character?
>
> <details><summary>Answer</summary>
>
> **Back at the original position.** Stopping play mode reverts the scene to the state it was in when you pressed Play. This is intentional — playtest changes shouldn't accidentally save into your scene. To make changes that persist, edit the scene *while not in play mode*.
>
> </details>

## Mapping tool (in-viewport, when you need it)

If you're authoring level geometry, the [Scene Mapping](../scene/scene-mapping/index.md) tool lives inside the viewport — its toolbar appears at the top of the 3D view when you select or create a `MeshComponent`. The mapping shortcuts (block tool, vertex / edge / face mode, texture mode, etc.) are all listed in [Scene Mapping Shortcuts](../scene/scene-mapping/shortcuts.md).

You don't need this on day one — most early projects use existing models, not custom geometry.

## Things that confuse newcomers

**The editor doesn't have a built-in code editor.** When you create a `.cs` Component in the Asset Browser, the file is created on disk but not opened anywhere. Open it manually in Visual Studio, Rider, or VS Code. The s&box editor watches the file system and recompiles on save.

**Hotload pauses on any compile error.** A red error in the Console means nothing else compiles until you fix it. When changes "stop applying," the Console is the first place to look.

**Stopping play mode reverts runtime changes.** Edits made *while playing* (positions, property values, spawned objects) don't persist when you press Stop. Edit while stopped if you want changes saved into the scene.

**Right-click is overloaded.** Right-click in the Scene Tree → context menu. Right-click in the viewport → context menu. Right-click + drag in the viewport → camera look. The action depends on whether you held the mouse before clicking.

## What to do next

Now you can find the panels, you're ready for [Your First Project](first-project.md) — make a spinning cube in five minutes. Most of the steps in that page reference panels described here; you'll know exactly where they live.

If you want to see a working scene to poke at instead, the [Explore the Engine](explore-engine.md) page points you at the Sandbox Testbed.

## See also

- [First Steps](first-steps/index.md) — the core concepts (what scenes / GameObjects / components even *are*).
- [Your First Project](first-project.md) — the first concrete build.
- [Editor Shortcuts](../editor/editor-shortcuts.md) — the full keybind reference.
- [Scene Mapping Shortcuts](../scene/scene-mapping/shortcuts.md) — only matters once you start authoring level geometry.
- [Editor (extending the editor)](../editor/index.md) — once you want to write *your own* editor tools, not just use the existing ones.
