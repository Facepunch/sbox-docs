---
title: "CodeGen"
icon: "⚙️"
sources:
  - engine/Tools/CodeGen/
created: 2026-04-27
updated: 2026-04-27
---

# CodeGen

This section covers the internal implementation of the `CodeGen` pipeline tool within the s&box engine.

## Architecture

This folder contains the standalone C# tool `CodeGen` used during the build phase of the engine. It parses all C# source code files in a given directory into syntax trees using Roslyn (`Microsoft.CodeAnalysis`), processes them through `Sandbox.Generator.Processor` to generate additional code (like serializers, networked property boilerplate, reflection information), and then writes the transformed `.cs` files into `obj/.generated`. These generated files are then compiled instead of the raw original files.

By using CodeGen as a pre-build step, s&box avoids using reflection heavily at runtime and handles networking seamlessly for the developer.

## Common Patterns

1. **Syntax Tree Generation:** `CodeGen` walks through source files, skipping build folders (`obj/`), and constructs Roslyn `SyntaxTree` objects.
2. **Processor Execution:** It feeds the `SyntaxTree`s into the engine's `Sandbox.Generator.Processor`, which analyzes classes, properties, and attributes (like `[Property]` or `[Broadcast]`).
3. **File Output:** It outputs the modified syntax trees (with the generated partial classes and methods) to `obj/.generated`, stamping them with a CRC based on the original file contents so MSBuild doesn't unnecessarily recompile unchanged files.

*Engine contributors modifying the engine-level Roslyn generators should test this tool locally, as it fundamentally dictates how game and engine C# code gets compiled.*
