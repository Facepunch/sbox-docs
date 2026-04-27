---
title: "Building from Source & Contributing"
icon: "🔨"
created: 2026-04-25
updated: 2026-04-25
sources:
  - Bootstrap.bat
  - CONTRIBUTING.md
  - engine/Sandbox-Engine.slnx
---

# Building from Source & Contributing

This is the page for an engine contributor: cloning, building, running, testing, and submitting a PR. If you just want to play games or make games using s&box, install through Steam — the prebuilt editor is what you want.

> **First time contributing?** This page is the reference. The [Engine Contributor Learning Path](learning-path-contributor.md) sequences it into a curriculum (Get oriented → First PR → Recurring contributions) — start there if you want structure, come back here for the specific commands.

## Prerequisites

Per the repo `README.md`:

- **Git**
- **Visual Studio 2026** (Rider works too; the engine uses standard `.csproj` projects)
- **.NET 10 SDK**
- **Windows 10/11** for development. Linux works for runtime via Proton; engine development is Windows-first.

## Clone

```bash
git clone https://github.com/Facepunch/sbox-public.git
cd sbox-public
```

## Build

The single command that builds everything:

```bat
Bootstrap.bat
```

What it actually runs (see `Bootstrap.bat`):

```bat
dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- build --config Developer
dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- build-shaders
dotnet run --project .\engine\Tools\SboxBuild\SboxBuild.csproj -- build-content
```

Three steps:

1. **`build --config Developer`** — compiles every `.csproj` in the engine solution in Developer config (debug-symbol-friendly, no optimisations).
2. **`build-shaders`** — runs the shader compiler over every shader in `game/core/shaders/` and `game/addons/*/shaders/`.
3. **`build-content`** — processes content (models, materials, sounds) into their compiled runtime forms.

A cold full bootstrap takes 5–15 minutes depending on machine. Subsequent incremental builds via `dotnet build` or your IDE's build are much faster.

After Bootstrap completes, the executables live under `game/bin/` (and `game/bin/win64/` for the native side).

## Run

```bat
game\bin\sbox-dev.exe
```

That's the editor. Other launcher variants in `game/bin/`:

| Executable | Purpose |
|---|---|
| `sbox.exe` | Player runtime (game launcher) |
| `sbox-dev.exe` | Editor |
| `sbox-server.exe` | Headless dedicated server |
| `sbox-standalone.exe` | Standalone-export player |
| `sbox-bench.exe` | Benchmark harness |
| `sbox-profiler.exe` | Profiler UI |

For most contributor work you're running `sbox-dev.exe`. Boot it once after a fresh build to confirm everything compiled correctly.

## Open the solution

```bat
engine\Sandbox-Engine.slnx
```

This is the canonical multi-project solution. Visual Studio 2026 and Rider both open `.slnx` natively. It contains every `Sandbox.X` project plus the `Tools/*` build-pipeline projects.

## Hotload while contributing

When modifying the **engine itself**, hotload doesn't help — you have to fully restart `sbox-dev.exe` to pick up engine-side changes. Hotload is for *user* code only.

When modifying user-facing content (an addon under `game/addons/`), hotload works the same as for any user. Save the file, the editor recompiles, your changes appear within milliseconds.

Practical workflow:

- **Engine change:** edit → build (`dotnet build` or IDE) → restart sbox-dev → test.
- **Addon change:** edit → save → look at the running editor for hotload result.
- **Native (`engine/bin/win64/*.dll`) change:** rebuild the native side → restart sbox-dev. The C# side may also need rebuilding if signatures changed.

## Tests

The test projects are under `engine/Sandbox.Test*/` and inside individual engine project directories.

### What's in each project

`Sandbox.Test/` — the main integration test project. Subdirectories map to the area under test:

| Subdir | Topic |
|---|---|
| `ActionGraphs/` | ActionGraph node serialization, runtime live-package scenarios |
| `Addon/` | Built-in addons, package and template loading |
| `Engine/` | Async tasks, audio, shutdown, Steam server tests |
| `Filesystem/` | Virtual filesystem, file watching |
| `Mounting/` | Mount system, custom mount providers |
| `MovieMaker/` | Movie binder, compiled movies, recording, scene playback |
| `Navigation/` | NavMesh generation, agent pathing |
| `Package/` | Package download, loader, code archive, package manager |
| `Physics/` | Mass, queries, collision rules |
| `Project/` | Project file loading and structure |
| `Scene/` | GameObjects (clone, colliders, component events/get/id, destroy, JSON upgraders), Components (model physics, polygon mesh serialization) |
| `ShaderGraph/` | Shader Graph compilation and node behaviour |
| `Standalone/` | Standalone-export build tests |
| `Texture/` | Texture loading and conversion |

`Sandbox.Test.Unit/` — focused unit tests for low-level systems. Subdirectories:

| Subdir | Topic |
|---|---|
| `Access/` | API whitelist / sandbox verification |
| `Bind/` | Native interop binding helpers |
| `BytePack/` | Binary serialization primitives |
| `Doo/` | DooEditor block-based logic |
| `Editor/` | Editor-only utility behaviour |
| `Events/` | Scene event interfaces (ISceneStartup, IScenePhysicsEvents, etc.) |
| `Json/` | JSON serialization helpers |
| `Math/` | Vector3/Rotation/Transform/BBox/Angles math |
| `Network/` | Networking primitives, snapshots, RPC dispatch |
| `Reflection/` | TypeLibrary catalogue |
| `SerializedObject/` | Property/object serialization for inspector |
| `Services/` | Backend services (achievements, leaderboards, stats) |
| `System/` | Sandbox.System utilities |
| `Time/` | Time scaling, fixed timestep |
| `UI/` | Razor panels, layout, world panels |
| `Web/` | HTTP/WebSocket client behaviour |

The per-system test projects ship inside the project they exercise:

| Project | What it tests |
|---|---|
| `Sandbox.Compiling.Test/` | The Roslyn-driven user-code compiler. `Tests/` contains compilation cases. |
| `Sandbox.CodeUpgrader.Test/` | Each upgrader rule's input → output behaviour. `Tests/` contains string-snippet test pairs. |
| `Sandbox.Hotload.Test/` | The deep-swap upgrader: delegate migration, member equality, type replacement, sync-var preservation. |
| `Sandbox.Test.Generate/` | Source generator output validation. |
| `Sandbox.Test.Hotload/` | Cross-hotload type-change scenarios (split into `BeforeHotload`/`AfterHotload` projects to simulate the swap). |

### Quick lookup: "where do I add a test for...?"

| Working on... | Add the test in... |
|---|---|
| A new built-in component property | `Sandbox.Test/Scene/Components/` |
| Networking / `[Sync]` / RPC | `Sandbox.Test.Unit/Network/` |
| Physics queries or behaviour | `Sandbox.Test/Physics/` |
| Hotload behaviour | `Sandbox.Hotload.Test/` (in-place) or `Sandbox.Test.Hotload/` (cross-swap) |
| Compiler / API whitelist | `Sandbox.Compiling.Test/` |
| Code upgrader rule | `Sandbox.CodeUpgrader.Test/Tests/` |
| Math types | `Sandbox.Test.Unit/Math/` |
| UI / Razor panels | `Sandbox.Test.Unit/UI/` |
| Mounting / external games | `Sandbox.Test/Mounting/` |
| Map system / scene loading | `Sandbox.Test/Scene/` |
| ActionGraph | `Sandbox.Test/ActionGraphs/` |
| Movie Maker | `Sandbox.Test/MovieMaker/` |
| NavMesh | `Sandbox.Test/Navigation/` |
| Filesystem / VFS | `Sandbox.Test/Filesystem/` |
| Source generator output | `Sandbox.Test.Generate/` |

Run the full suite from the solution root:

```bat
dotnet test engine\Sandbox-Engine.slnx
```

Or from inside an individual project:

```bat
cd engine\Sandbox.Test.Unit
dotnet test
```

Add tests when fixing bugs or adding features — `CONTRIBUTING.md` explicitly calls this out as expected.

## Code style

The repo uses an `.editorconfig` file at the root. Configure your IDE to honour it. Auto-format with:

```bat
dotnet format engine\Sandbox-Engine.slnx
```

Run this before committing if you've made formatting changes; CI runs the same check (`pull_request_formatting.yml`).

## Submitting a PR

The full workflow lives in `CONTRIBUTING.md`. The short version:

**Before you start:**

- For a new feature, file a proposal issue first. Discussion happens there.
- For a bug fix, reference the bug report number in your commit and PR.

**Branch & commits:**

- Branch off `main`.
- Group related changes per commit. Concise commit messages explaining *why*, not just *what*.
- Squash trivial commits before pushing.

**The PR itself:**

- Keep it tight. One concern per PR.
- Add tests where applicable (always for bug fixes, usually for features).
- Run `dotnet format` so the formatting check passes.
- Make sure all unit tests pass locally — CI will run them again.
- Pre-fill a clear summary: what was the problem, what was the fix, what does it affect.

**CI checks** (see `.github/workflows/`):

- `pull_request.yml` — full build + test
- `pull_request_checks.yml` — additional automated checks
- `pull_request_formatting.yml` — `dotnet format` check

All three need to be green before review. Review can ask for changes; iterate the same branch.

## Bug reports & security

Bugs go to [github.com/Facepunch/sbox-public/issues](https://github.com/Facepunch/sbox-public/issues), not in PRs. Include:

- Repro steps (numbered, exact)
- What you expected vs what happened
- Engine version (top-right of `sbox-dev`)
- Hardware / OS
- A minimal repro project if the bug is in something niche

Don't report bugs in community games here — those go to the game's author.

Security exploits get reported through [facepunch.com/security](https://facepunch.com/security), not GitHub.

## Where to start as a new contributor

If you're contributing for the first time and don't have a specific bug or feature in mind:

1. Read the [Architectural Overview](../architecture.md) to get the layer model.
2. Browse the [Where to Make Changes](where-to-make-changes.md) decision tree.
3. Look at the open issues tagged "good first issue" on GitHub.
4. Pick a bug fix rather than a feature for your first PR — fewer design questions.
5. When in doubt, ask in the proposal issue or on the Facepunch dev Discord.

## Related Pages

- [Where to Make Changes](where-to-make-changes.md) — the topic → project decision tree
- [Architectural Overview](../architecture.md) — the layered model
- [Engine Internals](index.md) — per-project deep dives
