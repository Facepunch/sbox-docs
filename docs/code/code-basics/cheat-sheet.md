---
title: "Cheat Sheet"
icon: "🐏"
created: 2024-08-06
updated: 2025-06-16
---

# Cheat Sheet

Sometimes you know what you're looking for, but you don't know where it is. The snippets below assume you're inside a `Component` (so `GameObject`, `Scene`, `Components`, `DebugOverlay`, etc. are accessible as inherited properties). Outside a `Component`, qualify with the relevant scope (`someGameObject.Components.Get<T>()`, etc.).

## Debugging

| Name | Code |
|------|------|
| Logging to console | `Log.Info( $"Hello {username}" );` |
| Warnings / errors | `Log.Warning( "..." );` &nbsp;·&nbsp; `Log.Error( "..." );` |
| Drawing text on screen *(in a Component)* | `DebugOverlay.ScreenText( new Vector2( 50, 50 ), "Hello" );` |
| Drawing a debug line in the world | `DebugOverlay.Line( from, to, color: Color.Red, duration: 2f );` |
| Drawing a debug sphere | `DebugOverlay.Sphere( center, radius, color: Color.Yellow );` |
| Asserting | `Assert.NotNull( obj, "Object was null!" );` |

> **Note:** `DebugOverlay` here is the inherited `Component.DebugOverlay` property (a `DebugOverlaySystem`), not a static class. From outside a Component, write `myComponent.DebugOverlay.X()`, `myGameObject.DebugOverlay.X()`, or `Scene.DebugOverlay.X()`.

## Time

| Name | Code |
|------|------|
| Frame delta (seconds) | `Time.Delta` |
| Total game time | `Time.Now` |
| Real time (ignores TimeScale) | `RealTime.Now` |
| Time since something happened | `TimeSince t = 0;` then `if ( t > 5f )` |

## Transforms

| Name | Code |
|------|------|
| Get GameObject world position | `var p = GameObject.WorldPosition;` |
| Set GameObject world position | `GameObject.WorldPosition = new Vector3( 10, 0, 0 );` |
| Get/set local position | `var p = GameObject.LocalPosition;` &nbsp;·&nbsp; `GameObject.LocalPosition = ...;` |
| World rotation | `GameObject.WorldRotation = Rotation.FromYaw( 90 );` |
| Whole transform at once | `var t = GameObject.WorldTransform;` |
| Convert local point → world | `var w = t.PointToWorld( localPoint );` |
| Convert world point → local | `var l = t.PointToLocal( worldPoint );` |

## GameObjects

| Name | Code |
|------|------|
| Find by name | `Scene.Directory.FindByName( "Cube" ).First();` *(returns IEnumerable)* |
| Find by Guid | `Scene.Directory.FindByGuid( guid );` |
| Creating | `var go = new GameObject();` &nbsp;·&nbsp; `var go = Scene.CreateObject();` |
| Cloning at a position | `var go = prefab.Clone( spawnPos );` |
| Deleting | `go.Destroy();` |
| Disabling | `go.Enabled = false;` |
| Adding a Tag | `go.Tags.Add( "player" );` |
| Has a tag (incl. inherited) | `go.Tags.Has( "player" )` |
| Iterate Children | `foreach ( var child in go.Children )` |
| Set parent | `child.SetParent( newParent );` |
| Validity check (after destroy) | `if ( go.IsValid() )` |

## Components

| Name | Code |
|------|------|
| Add component | `var c = GameObject.Components.Create<ModelRenderer>();` |
| Get if exists, else null | `var c = GameObject.Components.Get<ModelRenderer>();` |
| Get or add | `var c = GameObject.Components.GetOrCreate<ModelRenderer>();` |
| Remove component | `c.Destroy();` |
| Disable component | `c.Enabled = false;` |
| Get owner GameObject | `var go = c.GameObject;` |
| Iterate components on a GameObject | `foreach ( var c in GameObject.Components.GetAll() )` |
| All of a type, in self + descendants | `GameObject.Components.GetAll<PointLight>( FindMode.EverythingInSelfAndDescendants )` |
| All of a type in the whole scene | `foreach ( var c in Scene.GetAll<CameraComponent>() )` |
| Validity check | `if ( c.IsValid() )` |

## Input

| Name | Code |
|------|------|
| Held this frame | `Input.Down( "jump" )` |
| Pressed this frame | `Input.Pressed( "jump" )` |
| Released this frame | `Input.Released( "jump" )` |
| Move analog input | `Vector3 wish = Input.AnalogMove;` |
| Look analog input | `Angles look = Input.AnalogLook;` |

## Sound

| Name | Code |
|------|------|
| One-shot at a position | `Sound.Play( gunshotEvent, WorldPosition );` |
| Keep handle to control later | `var h = Sound.Play( ... ); h.Stop( fadeTime: 0.25f );` |
| Continuous, attached source | Use a `SoundPointComponent` instead of `Sound.Play` |

## Physics

| Name | Code |
|------|------|
| Trace a ray | `var tr = Scene.Trace.Ray( from, to ).Run();` |
| Sphere sweep | `Scene.Trace.Sphere( radius, from, to ).Run()` |
| Ignore self | `.IgnoreGameObjectHierarchy( GameObject )` |
| Apply impulse | `rigidbody.ApplyImpulse( Vector3.Up * 1000f );` |
| Apply continuous force | `rigidbody.ApplyForce( windDirection * strength );` |
