---
title: Sandbox.Compiling/CodeArchive
icon: "📦"
sources:
  - engine/Sandbox.Compiling/CodeArchive/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Compiling/CodeArchive

The `CodeArchive` module manages the storage, serialization, and retrieval of pre-compiled assemblies and their associated metadata.

## Common Patterns

1.  **Assembly Packing:** The compiler output is bundled into a structured archive format so the `.dll` and its `.pdb` (debug symbols) are kept strictly together, ensuring source-level debugging always works.
2.  **Checksum Validation:** Archives include hashing to verify that the compiled output matches the exact source state it was built from, preventing stale code loads.

## Related Pages
- [Sandbox.Compiling](sandbox-compiling.md)
