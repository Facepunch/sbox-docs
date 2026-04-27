---
title: "Crash Reporter"
icon: "🐛"
sources:
  - engine/Launcher/CrashReporter/
created: 2026-04-27
updated: 2026-04-27
---

# Crash Reporter

The `CrashReporter` is an external, minimal utility that runs alongside or after the engine process to catch unhandled exceptions, collect context (like logs and minidumps), and upload them for analysis.

## Quick Working Example

You generally do not write code for the Crash Reporter, as it runs as a separate process triggered upon a crash. However, here is a conceptual example of how the Crash Reporter is launched with a Sentry envelope.

```csharp
using System.Diagnostics;

// Inside the engine, upon catching a fatal crash:
public static void TriggerCrashReporter( string envelopePath )
{
	var processInfo = new ProcessStartInfo
	{
		FileName = "sbox-crash-reporter.exe",
		Arguments = $"\"{envelopePath}\"",
		UseShellExecute = false
	};

	Process.Start( processInfo );
}
```

## Common Patterns

### Managed Stack Extraction
A classic problem in mixed-mode engines is that a crash in native C++ code might have been triggered by a call from C#. A raw minidump shows the native transition but loses the C# context. The Crash Reporter actively hunts for recent `.mdmp` files and runs `ManagedStackExtractor.ExtractManagedStacks()` to append the C# frames as a text attachment.

### Envelope Packaging
The system uses an "Envelope" pattern (similar to Sentry's protocol) to bundle the main JSON event data alongside multiple raw attachments (logs, shader binaries, minidumps) into a single payload.

### Telemetry Redundancy
To ensure crash data isn't lost if one service is down, the Crash Reporter submits the payload to both a remote Sentry endpoint and a custom Facepunch telemetry API endpoint.

## Modifying the Crash Reporter

Engine contributors working on the Crash Reporter must be careful not to introduce dependencies on the main engine libraries (`Sandbox.Engine.dll`, etc.). The Crash Reporter must be able to boot and run completely independently. If it crashes while trying to report a crash, the telemetry is lost.

## Troubleshooting & Errors

:::warning Common Errors
- **Reporter fails to start:** Ensure the `sbox-crash-reporter.exe` executable is correctly packaged and placed alongside the main engine executable.
- **Missing Managed Stacks:** If the minidump file could not be generated or found by the `ManagedStackExtractor`, the crash report will only contain native C++ frames.
- **Aftermath Dumps Not Collected:** The reporter only collects `.nv-gpudmp` files generated within the last 120 seconds. If the crash reporter is delayed, these files might be skipped.
:::

## Related Pages
- [.NET Core Hosting](../../runtime-layout/launcher/netcore-hosting.md)
- [Engine Launcher](../launcher.md)
