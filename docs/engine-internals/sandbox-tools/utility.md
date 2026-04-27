---
title: "Editor Utility"
icon: "⚙️"
sources:
  - engine/Sandbox.Tools/Utility/Utility.cs
  - engine/Sandbox.Tools/Utility/UndoSystem.cs
  - engine/Sandbox.Tools/Utility/Utility.Prefabs.cs
created: 2026-04-27
updated: 2026-04-27
---

# Editor Utility & Core Systems

The `Sandbox.Tools.Utility` namespace provides the foundational systems that power editor workflows. This includes the Undo/Redo stack, global inspector state management, prefab serialization, and project publishing logic.

## Troubleshooting

:::warning
- **Broken Undo Chains:** If a custom editor tool begins an Undo block (`using var undo = new UndoBlock("Move")`) but an unhandled exception occurs inside the `using` scope, the state may become corrupted or partially applied. Always ensure code inside an undo block is resilient or wrapped in `try/catch` if it interacts with external files.
:::

## Related Pages
- [Editor Scene Management](scene.md)
