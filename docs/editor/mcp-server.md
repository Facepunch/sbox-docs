---
title: "MCP Server"
icon: "🤖"
created: 2026-07-06
updated: 2026-11-15
---

# MCP Server

The editor can run a [Model Context Protocol](https://modelcontextprotocol.io/) server. This lets AI agents like Claude Code see and drive the project you have open. An agent can read your scene, create and edit game objects, search assets, enter play mode, read the console, take screenshots, and anything else you expose as a tool.

It's on by default. Each editor exposes a local named pipe restricted to your operating system user. The `sbox-cli` command bridges this to MCP clients over stdin/stdout; there is no HTTP port to configure.

# Connecting an agent

Open your project in the editor and make sure **Project Settings → AI → Enable MCP Communication** is enabled. The `sbox-cli.exe` executable is in `bin/managed` inside your s&box installation.

## Claude Code

Run this in your project folder, replacing `<engine>` with your s&box installation path:

```
claude mcp add sbox -- "<engine>/bin/managed/sbox-cli.exe" --connect
```

Now start Claude Code from that project folder. The CLI connects to the editor for that project; `--connect` requires it to already be running. Closing the client leaves the editor open. Restart the client's MCP connection after restarting the editor.

## Any other client

Configure a stdio MCP server with the full path to `sbox-cli.exe` as its command and `--connect` as its argument. Set its working directory to your project folder, or pass the project's absolute path explicitly:

```
"<engine>/bin/managed/sbox-cli.exe" --connect "C:/projects/my-game"
```

## How agents find your tools

The client only sees a few entry points up front. Your tools are found live: the agent calls `search_tools` to find them and `call_tool` to run them. That's because tools come and go every time your code hotloads, so a fixed list would go stale. To test yours, ask the agent to `search_tools` for it.

# Writing your own tools

A tool is a static method. There's no registration. Write the method, save, and an agent can call it. It shows up and disappears as your code compiles.

```csharp
namespace Editor.Mcp;

[McpToolset( "car", "Everything about the cars in the scene" )]
public static class CarTools
{
	/// <summary>
	/// Find cars by name. Returns each car's id, which you pass to get_car for details.
	/// </summary>
	/// <param name="query">Case insensitive substring of the car's name. Empty matches everything.</param>
	/// <param name="limit">How many results to return. Default 20, max 100.</param>
	[McpTool( "find_cars" )]
	public static object FindCars( string query = "", int limit = 20 )
	{
		// ...find the cars...
		return new { Total = total, Cars = cars };
	}
}
```

That's it. The XML `<summary>` becomes the tool description and each `<param>` becomes a parameter description.

- **`[McpTool]`** marks the method. Pass a name (`"find_cars"`) or let one come from the method name in snake_case (`FindCars` becomes `find_cars`). Use **`[McpTool.ReadOnly]`** when the tool never changes anything. Clients can run those without asking the user first.
- **`[McpToolset]`** groups a class of tools under a name and description. Without it the name comes from the class name (`CarTools` becomes `car`).

:::info
Tool and toolset names are public API. Agents remember them, so their workflows break when you rename one. Pick good names and keep them.

:::

## The description is the API

Agents pick tools by reading the description. So say what the tool returns, what to call next, and anything surprising. Name other tools directly, like "the id you pass to `get_car`", so the agent knows where a value goes.

"finds cars" is a bad description. "Returns each car's id, which you pass to get_car" is a good one.

## Parameters

Parameters become the tool's JSON schema. A parameter with a default value is optional. Without one it's required.

| C# type | What the agent sends |
|---|---|
| `string` | A string. Anything else gets stringified, so this is the most forgiving type. |
| `bool` | true / false |
| `int`, `long`, `byte`... | An integer |
| `float`, `double`, `decimal` | A number |
| enums | The name as a string, case insensitive. The schema lists every value. |
| `Vector2/3/4`, `Vector2Int/3Int`, `Angles`, `Rotation` | A comma string, like `"100,0,50"` or `"pitch,yaw,roll"` |
| `Guid`, `DateTime`, `TimeSpan`, `char` | A string |
| `T[]`, `List<T>` | An array of T |
| `Dictionary<string, T>` | An object with T values |
| `JsonObject`, `JsonArray`, `JsonNode` | Raw json, taken as sent |
| A class or struct you wrote | An object (see below) |

Binding is loose on purpose, because agents make the same mistakes every time. Argument names match any case, numbers arrive as strings, JSON wrapped in a string gets unwrapped, wrong case enum names still work. A wrong argument name errors instead of being ignored, so a typo doesn't silently do the wrong thing.

## Custom classes and structs

A plain data type works as a parameter, or nested inside one. The schema comes from its public settable properties.

```csharp
/// <summary>
/// One waypoint on a patrol route.
/// </summary>
public class Waypoint
{
	/// <summary>Where to stand, as 'x,y,z'.</summary>
	public required Vector3 Position { get; set; }

	/// <summary>Seconds to wait before moving on.</summary>
	public float Wait { get; set; }
}

[McpTool( "set_patrol" )]
public static object SetPatrol( string id, Waypoint[] waypoints ) { /* ... */ }
```

- Every public property with a public setter becomes a schema property, sent in camelCase (`Position` becomes `position`). Binding ignores case, so both work.
- The `required` keyword makes a property required, and it's enforced. A call missing `position` errors.
- Describe properties with XML `<summary>` docs, same as everything else.
- `[JsonPropertyName]` and `[JsonIgnore]` are respected.

## Return values

Return whatever is most useful. It gets shaped into a result.

| Return | What the agent gets |
|---|---|
| `string` | The text as is |
| Any object | Serialized to json. Anonymous objects are the usual shape. |
| `GameObject` / `Component` | A short reference like `{ "type": "PointLight", "id": ..., "gameObject": ... }`, never the whole scene subtree. The id goes into `get_game_object`. |
| `Resource` (Model, Material, any GameResource) | Its path, like `"models/chair.vmdl"`, which goes into `asset_info`. |
| `Bitmap` | A PNG image |
| `McpResult` | Full control, so you can mix text and image blocks |
| `null` / `void` | "ok" |
| `Task<T>` / `async` | Awaited, then shaped as above |

Write the result for a reader, not a serializer. Return counts next to truncated lists (`new { Total, Showing, Cars }`) so the agent knows there's more, format dates as strings, and drop fields nobody needs. A small labelled result beats one big blob.

### Prefer a structured DTO

When a tool always returns the same shape, use a real class as the return type instead of `object`. Do this for anything an agent calls a lot.

```csharp
/// <summary>
/// A car and its current state.
/// </summary>
public class CarInfo
{
	/// <summary>The car's game object id. Pass to get_game_object.</summary>
	public Guid Id { get; set; }

	/// <summary>Display name.</summary>
	public string Name { get; set; }

	/// <summary>Current speed in units per second.</summary>
	public float Speed { get; set; }
}

[McpTool.ReadOnly( "get_car" )]
public static CarInfo GetCar( string id ) { /* ... */ }
```

A real class gives the agent an `outputSchema`. Before it even calls, it knows the exact fields, their types, and what each one means (from your XML docs), so it can plan around them instead of guessing from a response. The result also carries `structuredContent` the client can read straight off, instead of parsing text.

Anonymous returns (`new { ... }`) don't get a schema. They're fine for one-off tools. Use a real class when the shape matters and won't change.

### Returning images

Return a `Bitmap` and the agent gets a PNG it can actually look at. A screenshot, a thumbnail, a preview. Often more useful than describing the same thing.

```csharp
[McpTool.ReadOnly( "viewport_screenshot" )]
public static Bitmap ViewportScreenshot()
{
	var camera = Application.Editor?.Camera;
	if ( !camera.IsValid() )
		throw new Exception( "No editor scene camera. Is a scene viewport open?" );

	var bitmap = new Bitmap( 1280, 720 );
	camera.RenderToBitmap( bitmap, false );
	return bitmap;
}
```

For more control, return an `McpResult` and build the blocks yourself, like pairing a screenshot with a caption:

```csharp
return McpResult.Image( thumbnail ).WithText( $"Thumbnail for {asset.Path}" );
```

## Errors

Throw. The exception message reaches the agent as an error it reads and works around, so write it for that agent. Say what went wrong and what to do instead.

```csharp
throw new Exception( $"No car named '{name}'. Find cars with find_cars." );
```

Bad tool names, arguments that don't bind, anything thrown mid-tool: they all come back as readable text the agent can fix, not a protocol error.

## Threading

Tools run on the editor main thread, so you can touch scenes, assets and widgets directly. Tools can be `async`. Do slow work (network, compiles) after your last touch of engine state, or get back on the main thread first with `MainThread`.

# Conventions

A few rules apply to every tool, and they're fed to the agent automatically, so just follow them:

- Paging is `limit` and `offset`. Put the default and max in the parameter description.
- Vectors and angles are comma strings, like `'x,y,z'` or `'pitch,yaw,roll'`.
- Coordinates use the engine convention: one unit is one inch, +x forward, +y left, +z up, angles in degrees.
- Game objects and components use guids, assets use the path `asset_search` returns.
