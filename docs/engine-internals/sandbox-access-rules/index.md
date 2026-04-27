---
title: "Sandbox.Access/Rules"
icon: "📏"
sources:
  - engine/Sandbox.Access/Rules/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Access/Rules

The `Rules` module within `Sandbox.Access` contains the granular verification definitions applied to user assemblies during the IL inspection phase.

## Key Rule Categories

* **BaseAccess:** Governs access to foundational types from `System.Private.CoreLib` (like primitives, core collections like `List<T>`, basic math like `MathF`, and system delegates like `Action` or `Func`).
* **Types / Reflection:** Defines safe interactions with basic structural features. Importantly, `System.Reflection` is heavily restricted to prevent users from bypassing internal visibility modifiers or manipulating the engine memory space directly.
* **Numerics:** Exposes mathematically-intensive primitives and vectorized hardware instructions from `System.Numerics.Vectors`, specifically tailored to keep physics and intensive math logic efficient but secure. 
* **Exceptions / Diagnostics:** Whitelists standard errors (`InvalidOperationException`, `ArgumentNullException`) so game code can catch and throw basic structural faults without exposing dangerous host environment properties.
* **Async / CompilerGenerated:** Safelists state-machine generation types (`Task`, `ValueTask`, `TaskCompletionSource`, and specific `CompilerServices` markers). Due to how `async/await` transforms C# code structurally, these rules are crucial to allow modern async workflows without breaking the sandbox's threading or execution scope limits.

## IL Bytecode Verification Sequence

While `AccessRules` populates the allowed signatures, the actual checking happens inside `AssemblyAccess`. The sequence runs as follows:
1. `AssemblyAccess` uses `Mono.Cecil` to iterate over every `TypeDefinition` and `MethodDefinition` within the newly compiled assembly.
2. For every method's IL body, it inspects each instruction (`OpCodes`).
3. If an instruction references an external type or invokes an external method (e.g., `OpCodes.Call`, `OpCodes.Newobj`), `AssemblyAccess.Touch` is called with the target signature.
4. `Touch()` maintains a state of how many times a namespace/type has been hit, and verifies that signature against the compiled `AccessRules` regex evaluator.
5. If a signature matches an allowed rule pattern, the `AssemblyAccess` proceeds. If it is rejected, an error string and the specific IL instruction location are appended to the `AccessControlResult` blocking execution.

## Performance Profile

Because the rules are just definition files populating the primary engine Regex whitelist, they have virtually no overhead dynamically during runtime but heavily affect the initial assembly inspection step.

## Related Pages
- [Sandbox.Access](../sandbox-access.md)
