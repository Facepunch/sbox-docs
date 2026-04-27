---
title: "CreateGameCache"
icon: "⚙️"
sources:
  - engine/Tools/CreateGameCache/Program.cs
created: 2026-04-27
updated: 2026-04-27
---

# CreateGameCache

The `CreateGameCache` tool is a standalone pipeline application designed to pre-fetch and cache commonly used game assets (models, core maps, testbeds) from the backend API into a local `gamecache` folder.

## Common Patterns

1. **Manifest Processing:** For every identified package, the tool calls `DownloadManifestAsync()`. It then iterates through the files listed in the manifest and queues them for download.
2. **Asynchronous Throttling:** To avoid overwhelming the network interface or the backend API, the tool uses a `SemaphoreSlim` initialized with a limit of 16. This ensures that no matter how many thousands of files are queued, only 16 asynchronous download tasks (`Sandbox.Utility.Web.DownloadFile`) are executing concurrently.
3. **Delta Checking:** Before downloading a file, it checks if a file with the target cache name already exists in the `gamecache` folder. If it exists and the file size perfectly matches the size declared in the manifest (`info.Length == file.Size`), the download is skipped.

## Troubleshooting

:::warning
- **Missing Environment Variable:** If the `FACEPUNCH_ENGINE` user environment variable is not set correctly, the tool will attempt to resolve a path from `null` and crash when trying to create the cache directory.
- **API Rate Limiting:** If the tool fails to fetch packages, it may be due to authentication failure or rate limiting on the `Sandbox.Api` endpoints. Ensure you have a valid connection to the backend services.
:::

## Related Pages
- [Engine Launcher](../launcher.md)
