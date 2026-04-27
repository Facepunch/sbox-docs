---
title: Backend Services Architecture
icon: "☁️"
sources:
  - engine/Sandbox.Engine/Game/Services/
created: 2026-04-27
updated: 2026-04-27
---

# Backend Services Architecture

The Backend Services Architecture acts as the bridge between your running game and the s&box backend infrastructure, managing leaderboards, achievements, server listings, and cloud storage dynamically.

## Quick Start Example

As a game developer, you interact with the backend using the static `Sandbox.Services` API. For example, submitting a score:

```csharp
using Sandbox;
using Sandbox.Services;

public sealed class ScoreManager : Component
{
    public void SubmitScore( int finalScore )
    {
        // This is sent over an internal WebSocket or HTTP call to the Facepunch backend
        Leaderboards.Submit( "high_scores", finalScore );
        Log.Info( $"Submitted score {finalScore} to the backend." );
    }
}
```

## Common Patterns

1. **Fire-and-Forget Analytics:** Calls to increment stats or unlock achievements return `void`. They are considered "fire-and-forget". If the network fails, the local engine handles retries transparently.
2. **Asynchronous Fetching:** Methods that read data from the backend (like querying a leaderboard page) return `Task<T>`. You must `await` these in an async context, as network latency is variable.
3. **Session Tokens:** The `Auth` service automatically negotiates session tokens based on the user's Steam ID. You can request a token to verify a player's identity on your own custom web server if you need functionality beyond what the built-in services provide.

## Troubleshooting

:::warning
- **"HTTP 401 Unauthorized / Not logged in":** Happens if you are testing the game without Steam running, or if your developer token has expired.
- **"Missing Game Token":** Ensure your `.sbproj` is linked to an actual Organization and Project Identifier defined on `sbox.game`. If the engine doesn't know *which* game you are playing, it cannot resolve which leaderboard or achievement you are referencing.
:::

## Related Pages
- [Services Index](../index.md)
