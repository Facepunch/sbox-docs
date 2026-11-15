---
title: "Scripting"
icon: "📜"
created: 2026-11-15
updated: 2026-11-15
---

# Scripting

Scripting lets your game run C#-like source at runtime. Use it for editable expressions, player-authored logic, or [script assets](script-files.md) that define components and functions.

Scripts support a subset of C#, including expressions, loops, functions, classes, and properties, with access to game APIs.

## Running an expression

Use `Game.Scripting.Run<T>` for a short script that returns a value:

```csharp
var result = Game.Scripting.Run<float>( "return 12.0f * 1.5f;" );
Log.Info( result ); // 18
```

Use the matching return type. This call throws `Script.ScriptException` if the script fails, returns no value, or returns an incompatible type.

## Reusable scripts and arguments

Create a `Script` when you want to run the same source with different inputs. Pass values as named arguments instead of inserting them into the source string.

```csharp
var script = Game.Scripting.CreateScript( "return Strength * Multiplier;" );

var damage = script
    .With( "Strength", 10.0f )
    .With( "Multiplier", 1.5f )
    .RunAndReturn<float>();

if ( script.LastError is { } error )
{
    Log.Warning( error.ToString() );
}
else
{
    Log.Info( damage );
}
```

Argument names are case-sensitive and must be referenced by the script. Arguments can be values or objects.

You can also call `SetArgument` before running:

```csharp
script.SetArgument( "Strength", 20.0f );
script.SetArgument( "Multiplier", 2.0f );
var damage = script.RunAndReturn<float>();
```

Arguments are cleared after each run. Supply them again for the next run, and keep the `Script` instance when evaluating the same expression repeatedly.

### Checking compilation

Use `Compile` to check a script without executing it. Supply its arguments first:

```csharp
script.SetArgument( "Strength", 10.0f );
script.SetArgument( "Multiplier", 1.5f );

if ( !script.Compile() )
{
    Log.Warning( script.LastError.ToString() );
}
else
{
    var damage = script.RunAndReturn<float>();
}
```

Compilation leaves the arguments available for the next run.

## Keeping state

Use `[Local]` fields for values that should survive repeated runs. Each script gets its own local store by default. To preserve those values when replacing the source, create a store and pass it to both scripts:

```csharp
var state = Game.Scripting.CreateStore();

var counter = Game.Scripting.CreateScript( """
    [Local] int Count = 0;
    Count++;
    return Count;
    """, state );

Log.Info( counter.RunAndReturn<int>() ); // 1
Log.Info( counter.RunAndReturn<int>() ); // 2

var edited = Game.Scripting.CreateScript( """
    [Local] int Count = 0;
    Count += 10;
    return Count;
    """, state );

Log.Info( edited.RunAndReturn<int>() ); // 12
```

Stored fields initialize once by name. Use a new store to reset their values or change their types.

`[Global]` fields use `Game.Scripting.Globals`, shared across scripts. You can read and write stores from C# too:

```csharp
Game.Scripting.Globals.Set( "RoundNumber", 1 );
var round = Game.Scripting.Globals.Get<int>( "RoundNumber" );
```

`Get<T>( name )` requires an existing value of the matching type. `Get<T>( name, initialValue )` initializes a missing entry. Stores hold runtime state; they don't save values to disk or automatically synchronize them over the network.

## [Script Files](script-files.md)

Create `.csx` assets, define components, update named files, and share scripts with multiplayer clients.

