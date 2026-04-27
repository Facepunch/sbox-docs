---
title: "Playback API"
icon: "💻"
sources:
  - engine/Sandbox.Engine/Systems/Movies/MoviePlayer.cs
updated: 2026-04-25
created: 2026-04-27
---

# Playback API

The Playback API allows you to define, control, and play back cinematic movies and animations entirely through C# code.

## Quick Working Example

```csharp
using Sandbox;
using Sandbox.MovieMaker;

public sealed class MovieController : Component
{
	protected override void OnStart()
	{
		// 1. Create a reference track that binds to an object named "Camera"
		var objectTrack = MovieClip.RootGameObject( "Camera" );

		// 2. Create a property track to animate the camera's Field of View
		var fovTrack = objectTrack
			.Component<CameraComponent>()
			.Property<float>( "FieldOfView" )
			.WithSamples( timeRange: (1f, 3f), sampleRate: 2, new[] { 60f, 75f, 65f, 90f, 50f } );

		// 3. Group tracks into a clip
		var clip = MovieClip.FromTracks( fovTrack );

		// 4. Create a MoviePlayer to play the clip in the active scene
		var moviePlayer = GameObject.Components.Create<MoviePlayer>();
		moviePlayer.Clip = clip;
		
		// 5. Bind the reference track to our actual scene camera.
		//    The TrackBinder takes the IReferenceTrack you created above, not a string.
		moviePlayer.Binder.Get( objectTrack ).Bind( Scene.Camera.GameObject );

		// 6. Start playing!
		moviePlayer.Play();
	}
}
```

### Creating Constant Tracks

You can animate a property with constant values over specific time ranges.

```csharp
var positionTrack = objectTrack
    .Property<Vector3>( "LocalPosition" )
    .WithConstant( timeRange: (0.0, 2.0), new Vector3( 100, 200, 300 ) )
    .WithConstant( timeRange: (2.0, 5.0), new Vector3( 200, 100, -800 ) );
```

### Loading and Playing an Existing Movie Resource

Instead of creating clips in code, you can load a `.movie` file created in the editor and play it.

```csharp
var moviePlayer = Components.Get<MoviePlayer>();
moviePlayer.Resource = ResourceLibrary.Get<MovieResource>( "movies/intro_cutscene.movie" );
moviePlayer.Play();
```

### Finding Tracks in a Clip

`MovieClip.GetTrack(Guid)` looks up a reference track by ID. If you constructed the clip in code, hold onto the `IReferenceTrack` you created and pass it directly to the binder. If you loaded a clip from a `.movie` resource, iterate `clip.Tracks` and filter by what you need:

```csharp
// You built it — keep the reference track around
var camRef  = MovieClip.RootGameObject( "Camera" );
var clip    = MovieClip.FromTracks( /* tracks built from camRef */ );
moviePlayer.Binder.Get( camRef ).Bind( Scene.Camera.GameObject );

// You loaded it — walk the track list
foreach ( var track in clip.Tracks )
{
    if ( track is IReferenceTrack reference && reference.Name == "Camera" )
    {
        moviePlayer.Binder.Get( reference ).Bind( Scene.Camera.GameObject );
    }
}
```

There's no string-based "give me the property X on object Y" helper — the API works in terms of track instances and IDs, not paths.

### Creating Targets Automatically

By default, if the `MoviePlayer` cannot find a GameObject or Component referenced by the movie, it will create one for you. This is useful for spawning temporary effects or cameras.

```csharp
var moviePlayer = Components.Get<MoviePlayer>();

// Tell the player not to create missing targets automatically.
moviePlayer.CreateTargets = false;
moviePlayer.Play( clip );

// Force creation of any missing targets for the current Clip.
moviePlayer.UpdateTargets();
```

`UpdateTargets()` walks the active clip and creates missing GameObjects/Components for every reference track. The objects are parented under the `MoviePlayer`'s own GameObject and torn down on `OnDestroy`.

## Configuration

These properties are available on the `MoviePlayer` component:

| Property | Description |
|:---|:---|
| `Resource` | The `IMovieResource?` to play back. Accepts both `MovieResource` (asset on disk) and `EmbeddedMovieResource` (inline). |
| `Clip` | The underlying `IMovieClip?` — either `Resource.Compiled` or one assigned directly via code. |
| `Binder` | The `TrackBinder` used to resolve tracks to actual scene objects. Lazy-constructed on first access. |
| `IsPlaying` | Whether the movie is currently ticking. |
| `IsLooping` | If true, the movie restarts when it reaches the end. |
| `CreateTargets` | If true (default), automatically spawns missing GameObjects/Components referenced by tracks under the `MoviePlayer`'s GameObject. |
| `TimeScale` | Playback speed multiplier (`0..2`, default `1.0`). |
| `Position` | Current playback position as a `MovieTime` struct. |
| `PositionSeconds` | The current playback time as a `float` in seconds. Setter converts via `MovieTime.FromSeconds`. |

## Troubleshooting

:::danger "Track not animating my object!"
If your track isn't affecting the scene, it likely failed to bind. Make sure the name in your reference track (`MovieClip.RootGameObject("Camera")`) matches an object in your scene, or explicitly bind it using `Binder.Get( track ).Bind( myGameObject )`.
:::

:::warning "Movie creates unwanted clone objects"
If you play a movie and suddenly see duplicate models or cameras in your scene, it means `CreateTargets` is enabled and the `TrackBinder` couldn't find the original object. Ensure the binding paths match, or set `CreateTargets = false` to strictly prevent spawning new objects.
:::

## Related Pages
* [Getting Started](getting-started.md)
* [Recording API](recording-api.md)
