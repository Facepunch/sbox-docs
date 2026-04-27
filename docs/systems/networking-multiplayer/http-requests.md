---
title: "Http Requests"
icon: "🌐"
created: 2024-09-26
updated: 2026-04-13
sources:
  - engine/Sandbox.Engine/Utility/Web/Http.cs
  - engine/Sandbox.Engine/Utility/Web/Http.Requests.cs
---

# Http Requests

The static `Http` class allows your game to easily perform asynchronous web requests to external services, letting you fetch data, post high scores, or interact with REST APIs.

## Quick Working Example

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class MotdFetcher : Component
{
	protected override async Task OnLoad()
	{
		// Fetch a simple string from a website
		string motd = await Http.RequestStringAsync( "https://api.mygame.com/motd.txt" );
		Log.Info( $"Message of the Day: {motd}" );
	}
}
```

### Fetching and Parsing JSON

Instead of fetching a raw string and parsing it yourself, `RequestJsonAsync<T>` will automatically deserialize the JSON response into a C# object.

```csharp
public struct PlayerStats
{
	public int Kills { get; set; }
	public int Deaths { get; set; }
}

public async Task FetchStats( string playerId )
{
	var stats = await Http.RequestJsonAsync<PlayerStats>( $"https://api.mygame.com/stats/{playerId}" );
	Log.Info( $"Kills: {stats.Kills}, Deaths: {stats.Deaths}" );
}
```

### Sending Data (POST)

When you need to send data *to* a server (like submitting a score), you can use `RequestAsync` with the `POST` method. The `Http.CreateJsonContent<T>` method easily converts your C# objects into a JSON body payload.

```csharp
public async Task SubmitScore( string playerName, int score )
{
	var payload = new { Name = playerName, Score = score };
	
	// Convert the anonymous object to JSON content
	var content = Http.CreateJsonContent( payload );

	// Send the POST request
	await Http.RequestAsync( "https://api.mygame.com/leaderboard", "POST", content );
	Log.Info( "Score submitted successfully." );
}
```

### Passing Custom Headers

You might need to pass an API key or an authorization token to a secure endpoint.

```csharp
public async Task FetchSecureData()
{
	var headers = new Dictionary<string, string>
	{
		{ "Authorization", "Bearer my-secret-token" },
		{ "X-Game-Version", "1.2.0" }
	};

	string data = await Http.RequestStringAsync( "https://api.mygame.com/secure", headers: headers );
}
```

## Configuration / Restrictions

To prevent abuse and security vulnerabilities, the engine enforces a few rules on `Http` requests:

| Restriction | Description |
|---|---|
| **Domain Only** | You can only use `http` or `https` URLs pointing to domain names. Raw IP addresses are not permitted. |
| **Localhost Ports** | `localhost` requests are restricted to specific standard ports: `80`, `443`, `8080`, and `8443`. |
| **Local Network** | Requests to local network IPs (e.g. `192.168.x.x`) are generally blocked. |

:::info Bypassing Restrictions for Testing
If you are running a dedicated server or testing locally in the Editor, you can launch s&box with the command line switch `-allowlocalhttp`. This disables the local network restrictions, allowing you to test against your own local backend servers.
:::

## Troubleshooting

:::danger System.InvalidOperationException: Access to 'http://127.0.0.1/api' is not allowed.
The engine blocks requests to raw IP addresses and non-standard localhost ports. Use `localhost` on an allowed port (like `8080`), or start the engine with the `-allowlocalhttp` flag if you are developing locally.
:::

:::warning My request never finishes!
Make sure you are `await`ing the Http method call. Since web requests happen asynchronously, failing to `await` them means your code will continue executing immediately before the data arrives. Also, verify that the target server is actually responding.
:::

## Related Pages
- [WebSockets](websockets.md)
