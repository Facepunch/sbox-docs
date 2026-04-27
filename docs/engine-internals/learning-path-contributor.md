---
title: "Learning Path: Engine Contributor"
icon: "🎓"
created: 2026-04-25
updated: 2026-04-25
---

# Learning Path: Engine Contributor

A spiralled curriculum from "I just cloned the repo" to "I can pick up an arbitrary issue and ship a fix." Three iterations through the same core projects (`Sandbox.Engine`, `Sandbox.Hotload`, `Sandbox.Compiling`, `Sandbox.Reflection`, `Sandbox.Tools`) at increasing depth — each pass revisits them rather than meeting them once.

The pedagogy: a contributor who has read about hotload but never broken it doesn't really understand it. Each iteration forces a deeper encounter.

| Iteration | Goal | Time | What you can do at the end |
|---|---|---|---|
| 1. Get oriented | Build, run, navigate. Engine is a black box | ~half day | You can find a project by name and rebuild without consulting the README |
| 2. First PR | Touch one subsystem deeply enough to fix one thing | ~one weekend | Your name is in `git log` |
| 3. Recurring | Internalised understanding; pick up arbitrary issues | ongoing | You can triage a bug report and know which project it lives in |

Read sequentially. Each iteration assumes you finished the previous one.

---

## Iteration 1: Get oriented (~half day)

You'll clone the repo, get a clean build, run the test suite, run `sbox-dev.exe` from your local build, and skim the documentation that orients you for iteration 2. The point is **not** to understand the engine. It's to confirm you can build it, run it, and find your way around the source tree.

### 1.1 Clone and build (1 hour)
- [Building from Source & Contributing](contributing.md) — install prerequisites (.NET SDK, Visual Studio components, Steam install of s&box for the native binaries), run `Bootstrap.bat`, open `engine/Sandbox-Engine.slnx`.
- Build the solution from your IDE. **Build must succeed before you change anything.** If it doesn't, fix the environment now — debugging engine code on top of a half-broken build is misery.

After this you've seen: the project layout under `engine/`, the launcher executables (`sbox.exe`, `sbox-dev.exe`, `sbox-server.exe`), the `Bootstrap.bat` build flow. Don't worry about *what* the projects are — just notice they exist.

### 1.2 Run the test suite (15 min)
- `dotnet test engine\Sandbox-Engine.slnx` — should pass on a clean checkout.

If anything fails on a fresh clone, that's a real bug — file it before you continue. Tests in `Sandbox.Test.Unit/` are organised by topic (`Network/`, `Reflection/`, `Hotload/`, `Scene/`); skim the directory structure so you know where to add tests later.

### 1.3 Run your local `sbox-dev.exe` (15 min)
- Launch from your IDE with the `sbox-dev` target. Open any project.
- Edit one line in a user component, save, watch hotload swap it. **You're now using the engine you just built.**

This is the single most important moment of iteration 1: the engine you compiled is running, and code you compiled is hotloading user code. From here on, anything you change in `engine/` is something you can immediately try.

### 1.4 Read the orientation pages (1.5 hours)
Read in this order, as black-box reading — don't try to memorise:

- [Architectural Overview](../architecture.md) — the layer diagram. What's native, what's C#, what's the user's code.
- [Engine Concepts](../engine-concepts/index.md) — the prose tour. Boot sequence, runtime tick, hotload, VFS, sandboxing.
- [Engine Internals (this section's index)](index.md) — the per-project reference table. Bookmark this; you'll consult it constantly.
- [Where to Make Changes](where-to-make-changes.md) — the topic-to-project decision tree. **Don't memorise; bookmark.**

After this you've seen: `Sandbox.Engine`, `Sandbox.Hotload`, `Sandbox.Compiling`, `Sandbox.Access`, `Sandbox.Reflection`, `Sandbox.Tools`. Still as black boxes — you know they exist and roughly what each does.

### 1.5 Take one tour (30 min)
Pick one subsystem that interests you and skim its per-project page:
- [Sandbox.Engine](sandbox-engine/index.md) — the bulk of the gameplay API surface.
- [Sandbox.Hotload](sandbox-hotload.md) — the live-assembly-swap machinery.
- [Sandbox.Compiling](sandbox-compiling.md) — Roslyn integration and the analyser pipeline.
- [Sandbox.Reflection](sandbox-reflection.md) — `TypeLibrary` and the runtime metadata catalogue.
- [Sandbox.Tools](sandbox-tools/index.md) — the editor itself.

Open the project in your IDE alongside its docs page. Match the prose to the file structure. **Don't try to read every file.** You're looking for the shape of the project, not the contents.

### 1.6 What you should feel by now
- You can build the engine without consulting the README.
- `dotnet test` is muscle memory.
- You know which `.exe` to launch for which scenario.
- When someone says "Sandbox.Hotload," you know roughly which directory that is.
- You can read [Where to Make Changes](where-to-make-changes.md) and follow the decision tree without getting lost.

You don't yet know how the projects actually work, what the test patterns are, what review feedback looks like, or how to pick a real first task. That's iteration 2.

---

## Iteration 2: First PR (~one weekend)

Same projects, no longer black boxes. You'll pick a real "good first issue" and walk through find-fix-test-submit end-to-end. By the end your name is in `git log`.

### 2.1 Walk through one end-to-end (2 hours)
- [First Contribution Walkthrough](first-contribution-walkthrough.md) — a worked example fixing a real bug. Open this in a separate window and follow along on your own clone.

This is the linchpin of iteration 2. You're re-encountering the projects from iteration 1.5, but now in modify-mode: opening files, making changes, watching tests respond. By the time you finish the walkthrough you should know what "small change" means in this codebase.

### 2.2 Read the contribution conventions (1 hour)
- [Building from Source & Contributing](contributing.md) revisited — this time the **Submitting Changes** section, not the build instructions. Branch naming, commit-message style, PR description format.
- [Where to Make Changes](where-to-make-changes.md) revisited — now you're using it as a decision tool, not orientation reading. When the issue says "the inspector shows wrong default for `[Range]`," you should be able to land on `Sandbox.Tools` within seconds.

### 2.3 Pick a real first task (30 min)
Browse GitHub issues with the `good first issue` label. Pick something small and isolated — a typo in a public-facing string, a missing nullcheck, a property default that's wrong, an existing test you can extend. **Resist the urge to refactor.** A first PR exists to verify the round-trip, not to demonstrate taste.

If nothing on the issue tracker fits, look in the test suite for a `[Skip]`-marked test you can re-enable, or a `// TODO` you can resolve. Either is a fine first contribution.

### 2.4 Find the relevant project (30 min)
Use [Where to Make Changes](where-to-make-changes.md) to land on the right project. Open it in your IDE. Find the file you'll change.

This is where your iteration-1 black-box knowledge converts to working knowledge. The decision tree is now answering the question you actually have, not a hypothetical.

### 2.5 Make the smallest possible change (variable)
- Branch off `main`.
- Make the change.
- Save. If it's user-side test code, hotload picks it up; if it's engine code, rebuild.

For engine-side changes, you'll restart `sbox-dev.exe` to pick up the new engine assembly — the engine doesn't hotload itself. This is the first place iteration 1's knowledge of the launcher matters.

### 2.6 Test it (30 min)
- Add or extend a test in `engine/Sandbox.Test.Unit/`. The directory structure mirrors topics: networking changes go under `Network/`, reflection under `Reflection/`, etc.
- Run `dotnet test engine\Sandbox-Engine.slnx` and confirm green.

If you can't write a test for your change, that's a smell — either the change is too cosmetic to need one (typos), or the surrounding code is too tangled to test (in which case, smaller change first, refactor later).

### 2.7 Format and submit (30 min)
- `dotnet format engine\Sandbox-Engine.slnx` — non-negotiable.
- Commit with a clear message; reference the issue number.
- Push and open the PR. CI ([build](.github/workflows/pull_request.yml), [checks](.github/workflows/pull_request_checks.yml), [formatting](.github/workflows/pull_request_formatting.yml)) must all pass before review.

In the PR description: what was wrong, what your fix does, why this approach. **Three sentences is plenty.** Reviewers are reading dozens of these.

### 2.8 Iterate on review (variable)
Review feedback will come back. Read it carefully — reviewers in this codebase often surface conventions that aren't documented anywhere yet. Make the requested changes; push again; repeat. Don't argue points that aren't blocking your shipping.

### 2.9 What you should feel by now
- You've used [Where to Make Changes](where-to-make-changes.md) to land on the right project for a real task, not a hypothetical.
- You know what test patterns this codebase uses (`Sandbox.Test.Unit/`, the topic-based subdirectories, what good test names look like in this project).
- You've seen review feedback and know which kinds of comments are blocking and which are advisory.
- You've watched CI run on your branch.
- Your first PR is merged or in review.

You don't yet know the projects deeply enough to land on the right file before reading the docs. You don't have intuition for hotload behaviour. You haven't worked on the editor or written native interop. That's iteration 3.

---

## Iteration 3: Recurring contributions (ongoing)

By now you have a working understanding of the project layout. Iteration 3 is about deepening that model in the areas you contribute to most. There's no fixed time budget — pick the subsection that matches your next task.

### 3.1 The project you keep touching
Whichever project you find yourself in repeatedly, read its per-project page properly:

- [Sandbox.Engine](sandbox-engine/index.md) — for changes to `GameObject`, `Component`, lifecycle, the gameplay API surface itself.
- [Sandbox.Hotload](sandbox-hotload.md) — when you've broken hotload (you will) and need to understand why state migration failed.
- [Sandbox.Compiling](sandbox-compiling.md) — for analyser changes, Roslyn integration, the user-project build pipeline.
- [Sandbox.Reflection](sandbox-reflection.md) — for `TypeLibrary` consumers (the inspector, ActionGraph, the snapshot system all read it).
- [Sandbox.Tools](sandbox-tools/index.md) — for editor work: Inspector, Mapping, Hammer integration, Shader Graph, Movie Maker.
- [Sandbox.Bind](../engine-concepts/sandbox-bind.md) — for native interop changes; touches `.def` files and InteropGen.
- [Sandbox.Access](../engine-concepts/sandbox-access.md) — for whitelist changes or assembly-verification work.

Re-encountering these projects in third-pass depth means understanding their internal structure, not just their public surface. When you fix a bug in `Sandbox.Hotload`, you should know which file the deep-swap walker lives in before you open the project.

### 3.2 Hotload behaviour, internalised
The biggest gap between "I've contributed once" and "I can pick up arbitrary issues" is intuition for hotload. Read these once you have a contribution that touches type definitions, statics, or delegate caches:

- [Sandbox.Hotload](sandbox-hotload.md) — the deep-swap algorithm, type mapping, IL-patch fast path, the `OnRefresh` callback.
- [Hotloading developer guide](../code/code-basics/hotloading.md) — the user-facing pitfalls re-read from the engine side. The pitfalls *are* the engine-side bugs, viewed from the user's seat.

After a hotload-related fix you've shipped, you should know: when does state migration kick in vs. the IL-patch fast path? Why don't default field initialisers re-run? When does a cached `Type` reference prevent collection?

### 3.3 The boot path, when it matters
- [Runtime Layout](runtime-layout/index.md) — `engine/Launcher/`, `Startup.cs`, the AppSystem dependency graph.

You won't need this for most contributions. You'll need it if you're working on a launcher variant (server, dedicated, headless), changing AppSystem boot order, or debugging an issue that only manifests at startup.

### 3.4 The native interop layer
- [Sandbox.Bind](../engine-concepts/sandbox-bind.md), [Definitions](../engine-concepts/definitions.md), `engine/Tools/InteropGen/`.

Same caveat: only relevant if you're adding new native bindings. Most contributors never touch this layer. When you do, the path is `.def` file → InteropGen → generated C# → hand-written wrapper if needed.

### 3.5 Performance and profiling
- [Performance & Optimization](../systems/performance/index.md) revisited from the engine side. The `RemoteProf` overlay, profiler markers, allocation tracking.

For contributions that change hot-path code (the runtime tick, render submission, physics step), you need to be able to demonstrate that your change doesn't regress. Profile before, profile after, post the numbers in the PR.

### 3.6 Editor extensions
- [Sandbox.Tools](sandbox-tools/index.md) properly — Qt wrapping, Inspector internals, Hammer/Mapping/Shader Graph plumbing.
- [Editor](../editor/index.md) — the user-facing side of the same surface, useful as ground truth for what your editor changes affect.

Editor work is the largest single area of the engine and the one most divergent from gameplay code. If you find yourself contributing here repeatedly, the per-feature subpages under `sandbox-tools/` are where you'll live.

### 3.7 Mounting and external games
- [Mounting Internals](mounting.md), `engine/Mounting/Sandbox.Mounting.*`.

For supporting a new external game format. `BaseGameMount`, `Initialize`/`Mount`, on-demand asset conversion. This is rare, specialist work.

### 3.8 What you should feel by now
- A new bug report hits and you can guess which project before reading the report twice.
- You can read someone else's PR and tell whether their change is in the right place.
- You have intuition for which changes will break hotload and which won't.
- You're answering questions in PR review threads, not just receiving them.
- You can spot a missing test from the diff alone.

---

## Reference layer

These don't fit in any iteration. You consult them when you have a problem.

- [Engine Internals (per-project reference)](index.md) — the table of every `Sandbox.*` project with file paths.
- [Where to Make Changes](where-to-make-changes.md) — the topic-to-project decision tree.
- [Architectural Overview](../architecture.md) — the layer diagram, when you need to re-orient.
- [Engine Concepts](../engine-concepts/index.md) — the prose tour, when you've forgotten how a subsystem fits in.
- [First Contribution Walkthrough](first-contribution-walkthrough.md) — re-readable as a worked example.

## Maintainer notes

The pedagogy: this is **spiralled, not linear** (Bruner). The same projects appear in iterations 1, 2, and 3 at increasing depth — black-box → modify-mode → internalised. If you edit the curriculum, preserve that — don't introduce a project once and never return.

The contributor curriculum mirrors the [game-dev learning path](../learning-path-game-dev.md) deliberately. A reader who has done one will recognise the shape of the other; that's the point.

Each step's prose hook should answer three questions: *Why this page now?* What does the reader already know; what does this add? *What's the actionable thing they'll do?* Build, fix, profile, submit? *When can they move on?* What does "I get this" feel like — the success criterion that means they're ready for the next step.
