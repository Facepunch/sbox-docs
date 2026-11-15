---
title: "Script Files"
icon: "📄"
created: 2026-11-15
updated: 2026-11-15
---

# Script Files

`.csx` files are plain-text script assets. Put them in your project's `Assets` folder, for example `Assets/scripts/spinning-sign.csx`. Changes are loaded automatically.

## Defining a component

A script file can define a component with properties and lifecycle methods:

```csharp
/// <summary>
/// Rotates a sign around its vertical axis.
/// </summary>
public class SpinningSign : Component
{
    /// <summary>
    /// Rotation speed in degrees per second.
    /// </summary>
    [Property]
    public float Speed { get; set; } = 45.0f;

    /// <summary>
    /// Applies the rotation each frame.
    /// </summary>
    protected override void OnUpdate()
    {
        WorldRotation *= Rotation.FromYaw( Speed * Time.Delta );
    }
}
```

Script components can be added through the editor. `[Property]` exposes values in the inspector. Compatible edits hotload the component and preserve its existing state.

To use `[Sync]` or RPC attributes, put them on a regular C# component base and derive your script component from it.

## Startup code

Statements outside a function or class run when the file starts:

```csharp
Log.Info( "Loaded the sign scripts." );
```

Files can refer to types and functions declared in other script files. Startup code can run again when scripts reload, so account for this when creating objects or subscribing to events.

## Loading source from code

Use `LoadFile` to add or replace one named script while keeping the other files:

```csharp
var result = Game.Scripting.LoadFile(
    "scripts/greeting.csx",
    "Log.Info(\"Hello from a script\");"
);

if ( !result.Success )
{
    foreach ( var diagnostic in result.Diagnostics )
    {
        Log.Warning( diagnostic.ToString() );
    }
}
```

This updates the running game without writing a file to disk. Use `RemoveFile( "scripts/greeting.csx" )` to remove its declarations. Scripts loaded from disk can be restored on a later reload.

### Updating a complete file set

`LoadFiles` replaces the complete named-file set. Include every file you want to retain: omitted files and their declarations are removed. Use this when your game owns a whole collection of scripts, rather than for a one-file edit.

```csharp
var result = Game.Scripting.LoadFiles( new Dictionary<string, string>
{
    ["scripts/rules.csx"] = "int GetRoundLimit() { return 10; }",
    ["scripts/startup.csx"] = "Log.Info(GetRoundLimit());"
} );
```

Check `Success` and `Diagnostics` after loading. If `RequiresReload` is set, restart the game before running more scripts. Startup errors don't undo changes the script has already made.

## Multiplayer

The host supplies the script files. Ordinary `.csx` files, updates, and removals are replicated to clients, including players who join later.

Use a `.server.csx` suffix for a file that should execute only on the original host, such as `scripts/rounds.server.csx`.

:::info
The server-only suffix controls execution and replication. A published package still contains the source file, so it doesn't keep that source confidential.
:::

Shared startup code runs on clients too. Script variables aren't automatically synchronized; use normal networking and ownership checks for game actions.
