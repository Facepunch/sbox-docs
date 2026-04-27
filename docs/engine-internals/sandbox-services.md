---
title: "Sandbox.Services"
icon: "☁️"
sources:
  - engine/Sandbox.Services/
created: 2026-04-27
updated: 2026-04-27
---

# Sandbox.Services

The `Sandbox.Services` namespace contains the low-level HTTP clients, WebSocket connectors, and Protobuf serialization models used to communicate with the primary s&box backend (sbox.game).

## Quick Start Example

As an engine contributor working on new backend features, you might write a new low-level API call:

```csharp
using Sandbox.Services;

internal class BackendDebugger
{
    public async Task FetchAddonInfo( string ident )
    {
        // Conceptual: Engine level request to the asset.party backend
        var request = new ApiRequest( $"games/{ident}" );
        var response = await request.SendAsync<PackageData>();
        
        if ( response != null )
        {
            Log.Info( $"Fetched info for {response.Title}" );
        }
    }
}
```

## Common Patterns

1. **Batching & Queuing:** To prevent spamming the backend, this layer often batches multiple small requests (like 10 different stat updates in one second) into a single HTTP payload.
2. **Retry Policies:** Network instability is common. The API requests here often implement exponential backoff retry logic if the backend responds with a `502` or `503` error.

## Visual Diagnostics

In the s&box Editor, you can monitor live requests made by `Sandbox.Services` using the internal Network Profiler tool. This shows HTTP status codes, payload sizes, and exact latency timings for every call made to the asset.party backend.

## Troubleshooting

:::warning
- **"HTTP 429 Too Many Requests":** If an engine developer writes a tight loop that calls the backend (e.g., trying to fetch 1000 player avatars sequentially instead of batching them), the backend will rate-limit the client.
- **"Failed to serialize payload":** If the C# model and the backend Protobuf definition fall out of sync, the backend will reject the request. Ensure the proto schemas match the live API.
:::

## Sample Validation
Look at the integration tests within `engine/Sandbox.Test/` that mock the HTTP client to ensure `Sandbox.Services` correctly handles unexpected `500 Internal Server Error` responses without crashing the engine.

## Related Pages
- [Backend Services Architecture](../systems/services/architecture/backend.md)
