---
title: "Console Variables"
icon: "🔣"
created: 2025-01-08
updated: 2025-06-15
---

# Console Variables

You can create variables and commands that you can run from the console.

# Commands

Console commands are just static methods with an attribute. Here running `hello` in the console will cause it to print `Hello There!`

```csharp
    [ConCmd("hello")]
    static void HelloCommand()
    {
        Log.Info( "Hello there!" );
    }
```

Commands can also have arguments. Here running `hello dave` will print out `Hello there dave!`. 

```csharp
    [ConCmd("hello")]
    static void HelloCommand( string name )
    {
        Log.Info( $"Hello there {name}!" );
    }
```

:::info
The backend will try its best to convert the arguments from strings to the type you specify.

:::

## Server Commands

You might want to have a command that is always run on the server. You can do this by adding the `ConVarFlags.Server` flag to the `ConCmd` attribute.

:::tip
If you make the first parameter of the method have the `Connection` type, you'll be able to tell who actually ran the command.

:::

```csharp
	[ConCmd( "test", ConVarFlags.Server )]
	public static void TestCmd( Connection caller )
	{
		Log.Info( "The caller is: " + caller.DisplayName  );
	}
```

# Variables

It can useful to have variables that you can tweak to configure certain elements of your game. 

Here's an example:

```csharp
	[ConVar]
	public static bool debug_bullets{ get; set; } = false;
```

ConVars can have flags. These can be combined.

```csharp
//
// Is saved to disk and restored, so it persists between sessions
//
[ConVar( "bullet_count", ConVarFlags.Saved )]
public static int BulletCount { get; set; } = 6;

//
// Only the host can change it, and its value is synced to all clients
//
[ConVar( "friendly_fire", ConVarFlags.Replicated )]
public static bool FriendlyFire { get; set; } = false;

//
// Is sent to the host in UserInfo, which is available via the client's Connection
//
[ConVar( "view_mode", ConVarFlags.UserInfo )]
public static string ViewMode{ get; set; } = "firstperson";

//
// Hide in find and autocomplete. This works on ConCmd too.
//
[ConVar( "secret", ConVarFlags.Hidden )]
public static int SecretVariableMode { get; set; } = 3;
```

## Game Settings

You can use `ConVarFlags.GameSetting` to expose a setting to the game's creation screen, useful for configuring a game.

```csharp
//
// Shows up in the game creation screen, as a slider.
//
[ConVar( "player_speed", ConVarFlags.GameSetting ), Range( 50f, 1024f, 1 )]
public static float PlayerSpeed { get; set; } = 250f;
```
