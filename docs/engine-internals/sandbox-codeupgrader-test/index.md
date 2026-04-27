---
title: "Sandbox.CodeUpgrader.Test"
icon: "🔬"
sources:
  - engine/Sandbox.CodeUpgrader.Test/
updated: 2026-04-25
created: 2026-04-27
---

# Sandbox.CodeUpgrader.Test

The `Sandbox.CodeUpgrader.Test` assembly contains the automated test suite responsible for validating the Code Upgrader rules against real-world C# snippets.

## Quick Working Example

A typical upgrader test creates an isolated compilation unit, runs a specific upgrader, and verifies the rewritten source code string exactly matches the expected output.

```csharp
using Sandbox.CodeUpgrader;
using Sandbox.CodeUpgrader.Test;

[TestClass]
public class VectorDistanceTests : UpgraderTestBase
{
    [TestMethod]
    public void TestDistanceToRename()
    {
        var input = @"
            class Test {
                void Run() {
                    float dist = a.DistanceTo(b);
                }
            }
        ";

        var expected = @"
            class Test {
                void Run() {
                    float dist = a.Distance(b);
                }
            }
        ";

        // Validate that running VectorDistanceUpgrader produces the expected output
        AssertUpgrades( new VectorDistanceUpgrader(), input, expected );
    }
}
```

## Common Patterns

1. **AssertUpgrades:** This is the primary verification method used across the test suite. It abstracts away the boilerplate of compiling the `input` string with Roslyn, executing the provided `SyntaxRewriterUpgrader`, and asserting the resulting formatted text against the `expected` string.
2. **Trivia Validation:** Many tests explicitly include comments (`// trailing comment`) in the input string to ensure that the upgrader rule properly uses `WithTriviaFrom` to preserve formatting during the node replacement.

## Troubleshooting

:::warning
- **Whitespace Mismatches:** Roslyn is extremely strict about whitespace formatting. A test might fail not because the logic was upgraded incorrectly, but because the expected string has differing indentation or trailing spaces compared to the generated output. Ensure test cases exactly mirror the upgrader's formatting behavior.
:::

## Related Pages
- [Sandbox.CodeUpgrader](../sandbox-codeupgrader/index.md)
