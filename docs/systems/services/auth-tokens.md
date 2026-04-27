---
title: "Auth Tokens"
icon: "🔑"
created: 2023-12-22
updated: 2025-08-14
---

# Auth Tokens

If you are using HTTP requests or [WebSockets](../networking-multiplayer/websockets.md) in your game, you can use Auth Tokens to validate that the requests were sent from a valid Steam user in a s&box game session. This is useful if you want to tie data to a specific Steam account, or prevent botting.

# Generating Tokens

You can generate a new token with Sandbox.Services like so:

```csharp
var token = await Sandbox.Services.Auth.GetToken( "YourServiceName" );
```

# Validating Tokens

To validate a token on your backend, you need to make a call to the `services.facepunch.com/sbox` API using the `auth/token` endpoint.

Here is an example of how to validate a token in C# using `System.Net.Http`

```csharp
private class ValidateAuthTokenResponse
{
	public long SteamId { get; set; }
	public string Status { get; set; }
}

public static async Task<bool> ValidateToken( long steamId, string token )
{
	var http = new System.Net.Http.HttpClient();
	var data = new Dictionary<string, object>
	{
		{ "steamid", steamId },
		{ "token", token }
	};
	var content = new StringContent( JsonSerializer.Serialize( data ), Encoding.UTF8, "application/json" );
	var result = await http.PostAsync( "https://services.facepunch.com/sbox/auth/token", content );

	if ( result.StatusCode != HttpStatusCode.OK ) return false;
	
	var response = await result.Content.ReadFromJsonAsync<ValidateAuthTokenResponse>();
	if ( response is null || response.Status != "ok" ) return false;

	return response.SteamId == steamId;
}
```

At some point when receiving the token from the client on your backend you can then validate it as such:

```csharp
var isValidToken = await ValidateToken( steamId, token );

if ( isValidToken )
{
	Console.WriteLine( "Success!" );
}
```
