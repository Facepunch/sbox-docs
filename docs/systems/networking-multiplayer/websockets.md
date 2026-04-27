---
title: "WebSockets"
icon: "🌍"
created: 2024-09-26
updated: 2026-04-13
sources:
  - engine/Sandbox.Engine/Utility/WebSocket.cs
---

# WebSockets

Connect your game to external servers to send and receive real-time text or binary messages using the `WebSocket` class.

## Quick Working Example

```csharp
using Sandbox;
using System.Threading.Tasks;

public sealed class MyWebSocketClient : Component
{
	private WebSocket _socket;

	protected override async Task OnLoad()
	{
		// 1. Create a new WebSocket instance
		_socket = new WebSocket();

		// 2. Listen for incoming text messages
		_socket.OnMessageReceived += ( message ) =>
		{
			Log.Info( $"Server says: {message}" );
		};

		// 3. Connect to the server
		await _socket.Connect( "wss://echo.websocket.org" );

		// 4. Send a message
		await _socket.Send( "Hello, external world!" );
	}
	
	protected override void OnDestroy()
	{
		// Always dispose of the socket when you are done to free up connections
		_socket?.Dispose();
	}
}
```

### Connecting with Auth Tokens

If you are communicating with a secure server, you can pass custom HTTP headers (like Authorization tokens) when connecting.

```csharp
public async Task ConnectWithAuth()
{
	var token = await Sandbox.Services.Auth.GetToken( "YourServiceName" );

	if ( string.IsNullOrEmpty( token ) )
	{
		Log.Warning( "Unable to fetch a valid session token." );
		return;
	}

	var headers = new Dictionary<string, string>()
	{
		{ "Authorization", token }
	};

	_socket = new WebSocket();
	await _socket.Connect( "wss://your-api.com/ws", headers );
}
```

### Handling Binary Data

If your server sends binary data instead of text (for example, raw protobufs or custom packet structures), you should listen to the `OnDataReceived` event instead.

```csharp
_socket.OnDataReceived += ( Span<byte> data ) =>
{
	Log.Info( $"Received {data.Length} bytes of binary data." );
	// Process your binary data here
};

// Sending binary data
byte[] myBytes = new byte[] { 0x01, 0x02, 0x03 };
await _socket.Send( myBytes );
```

### Handling Disconnections

You can hook into the `OnDisconnected` event to handle automatic reconnects or notify the player that the connection to the backend was lost.

```csharp
_socket.OnDisconnected += ( int status, string reason ) =>
{
	Log.Info( $"Disconnected! Status: {status}, Reason: {reason}" );
};
```

The `WebSocket` class provides the following properties to manage its state:

| Property | Type | Description |
|---|---|---|
| `IsConnected` | `bool` | Returns `true` if the socket is currently open and connected. |
| `EnableCompression` | `bool` | Whether to enable per-message deflate compression. Default is `true`. |
| `SubProtocol` | `string` | The negotiated sub-protocol with the server, if any. |

You can also restrict the maximum message size by passing it into the constructor (e.g. `new WebSocket( maxMessageSize: 1024 * 1024 )`). The default max size is 64 KB.

## Troubleshooting

:::warning "System.ObjectDisposedException: Cannot access a disposed object"
Make sure you aren't trying to call `_socket.Send()` or `_socket.Connect()` after `_socket.Dispose()` has been called, or after the Component has been destroyed.
:::

:::danger Connection Refused
If `Connect()` throws an exception or fails silently, ensure you are connecting to a valid `ws://` or `wss://` URI. WebSockets cannot connect to standard HTTP endpoints that don't support the upgrade protocol.
:::

## Related Pages
- [Http Requests](http-requests.md)