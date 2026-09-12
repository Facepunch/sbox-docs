---
title: "Playback API"
icon: "code"
created: 2025-06-30
updated: 2026-09-03
---

# Playback API

You can define movies and play them back using C#.

# Tracks

Tracks can represent a GameObject reference, a Component reference, or a property that should be animated.

Create a reference track that, when played, binds to an object named "Camera".

```csharp
using Sandbox.MovieMaker;
using Sandbox.MovieMaker.Compiled;

var objectTrack = MovieClip.RootGameObject( "Camera" );
```

Create a property track that binds to a property named "LocalPosition" inside of whatever *objectTrack* is bound to.

Give it a constant value between 0s and 2s, then a different constant value from 2s to 5s. After 5s it has no value.

```csharp
var positionTrack = objectTrack
    .Property<Vector3>( "LocalPosition" )
    .WithConstant( timeRange: (0.0, 2.0), new Vector3( 100, 200, 300 ) )
    .WithConstant( timeRange: (2.0, 5.0), new Vector3( 200, 100, -800 ) );
```

Create a property track that binds to the "FieldOfView" property of a *CameraComponent* attached to whatever *objectTrack* is bound to. Give it an array of samples between 1s-3s.

```csharp
var fovTrack = objectTrack
    .Component<CameraComponent>()
    .Property<float>( "FieldOfView" )
    .WithSamples( timeRange: (1f, 3f), sampleRate: 2, [60f, 75f, 65f, 90f, 50f] );
```

Animate the scene with each track. They will try to bind to root objects in the scene with the matching names and component types, and silently fail if they don't exist.

```csharp
var update = new MovieUpdateBuilder();

update.Add( positionTrack, time );
update.Add( fovTrack, time );
update.Apply();
```

# MovieClip

Group tracks into a clip so we can animate them all together.

```csharp
var clip = MovieClip.FromTracks( positionTrack, fovTrack );

clip.Update( time );
```

You can search for tracks by path.

```csharp
var camTrack = clip.GetReference<GameObject>( "Camera" );
var posTrack = clip.GetProperty<Vector3>( "Camera", "LocalPosition" );
```

Clips can be serialized to and from JSON.

```csharp
Log.Info( Json.Serialize( clip ) );
```

# TrackBinder

What does *objectTrack* bind to in the current scene?

```csharp
var target = TrackBinder.Default.Get( objectTrack );

Log.Info( target.IsBound );
Log.Info( target.Value );
```

```
12:59:08 Generic  True
12:59:08 Generic  GameObject:Camera
```

Bind the track to something else.

```csharp
target.Bind( Game.ActiveScene.Camera.GameObject );
```

Binders can be serialized too.

```csharp
Log.Info( Json.Serialize( TrackBinder.Default ) );
```

We can have multiple *TrackBinder* instances, so the same clip can control different objects.

```csharp
var binder = new TrackBinder( Game.ActiveScene );

binder.Get( objectTrack ).Bind( Game.ActiveScene.Camera.GameObject );

// Using TrackBinder.Default

clip.Update( time );

// Using our own TrackBinder instance

clip.Update( time, binder );
```

## Target Creation

By default, the player will create any *GameObject*s or *Component*s referenced by the recording that aren't already in the scene. These targets will be flagged as *NotSaved | NotNetworked*. You can turn this off with the *CreateTargets* property.

```csharp
moviePlayer.CreateTargets = false;
moviePlayer.Play( clip );
```

You can also manually create targets through the player's *TrackBinder*.

```csharp
// Create targets for every track in the given clip
moviePlayer.Binder.CreateTargets( clip );

// Create targets for a specific set of tracks
var track1 = clip.GetReference<ModelRenderer>( "Example", "ModelRenderer" );
var track2 = clip.GetReference<PlayerController>( "Player", "PlayerController" );

moviePlayer.Binder.CreateTargets( [track1, track2] );
```

# MoviePlayer

The MoviePlayer component has a clip, a binder, and a time position.

```csharp
var moviePlayer = GameObject.AddComponent<MoviePlayer>();

moviePlayer.Clip = clip;
moviePlayer.Binder.Get( clip.GetReference<GameObject>( "Camera" ) ).Bind( Game.ActiveScene.Camera.GameObject );

// Time in seconds

moviePlayer.Position = 0.75;
```

The clip can also be from a resource.

```csharp
moviePlayer.Resource = ResourceLibrary.Get<MovieResource>( "example.movie" );
```
