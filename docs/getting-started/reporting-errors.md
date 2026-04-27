---
title: "Reporting Errors"
icon: "🚨"
created: 2024-01-18
updated: 2025-06-15
sources:
  - engine/Launcher/CrashReporter/
---

# Reporting Errors

Errors will happen. Here's how to make a useful error report so we can fix bugs in the engine quickly.

## Quick API Example: Custom Error Reporting

If you are developing a game and want to log custom information to the console before an error occurs, you can use the standard `Log` class. This output will be included in the player's log file if they need to report an issue to you.

```csharp
using Sandbox;

public sealed class ErrorLogger : Component
{
    protected override void OnStart()
    {
        try
        {
            // Simulate risky operation
            int.Parse("not a number");
        }
        catch ( System.Exception e )
        {
            // Log.Error will print to the console in red and write to the log file
            Log.Error( e, "Failed to parse vital configuration data!" );
        }
    }
}
```

- **"Silent" Failures:** Sometimes logic fails without crashing or logging an error (e.g., an `OnUpdate` loop stops). This usually indicates an object was silently disabled or destroyed. Use `Log.Info` to trace the execution path before assuming it's an engine bug.

## Related Pages

* [Getting Started Index](index.md)
* [FAQ](faq.md)
