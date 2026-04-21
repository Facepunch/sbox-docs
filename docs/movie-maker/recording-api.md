---
title: "Recording API"
icon: "code"
created: 2026-02-04
updated: 2026-03-18
---

# Recording API

You can record gameplay into a *MovieClip*, which can be played back or imported into Movie Maker for editing. This could be useful for killcams in shooters, leaderboard replays in racing games, or for capturing gameplay to edit into a trailer.

This feature is provided by the *MovieRecorder* class. You can use its default behaviour to capture all display-related components in the scene, or configure it to capture only certain objects, components and properties.

# Recording Basics

By default, a *MovieRecorder* will capture all *Renderer*s, *Camera*s, *SoundPoint*s and particle-related components. Just create a recorder, and call *Start*.

```csharp
// Create a recorder using the default capturing options
var recorder = new MovieRecorder( Scene );

// Start capturing
recorder.Start();
```

This starts capturing the scene every fixed update. When you want to stop recording, call *Stop*.

```csharp
// Stop capturing
recorder.Stop();
```

You can then compile the recording into a *MovieClip* for playback in a *MoviePlayer*, or to save into a *MovieResource*.

```csharp
// Convert the recording to a MovieClip
var clip = recorder.ToClip();

// Play the clip!
GetComponent<MoviePlayer>().Play( clip );

// Save it to a .movie
FileSystem.Data.WriteJson( $"movies/example.movie", clip.ToResource() );
```

## Manual Capture

In some cases you'll need more control over when captures are taken, like when recording in-editor. Instead of *Start* and *Stop*, call *Advance* to move the recording along by an amount of time, and *Capture* to record a frame.

```csharp
while ( IsRecording )
{
    SimulateScene();

    // Move the recording time along by 10ms
    recorder.Advance( 0.01 );
    
    // Capture a frame
    recorder.Capture();
}
```

# Configuring Recorders

If you want to be more selective about what gets recorded, create a *MovieRecorderOptions* instance and pass it in to the *MovieRecorder* constructor. These options let you specify the sample rate, filter which *GameObject*s are allowed to be captured, and add actions to run during each capture.

A new options instance won't capture anything, letting you fully configure it.

```csharp
var options = new MovieRecorderOptions( SampleRate: 60 )
    .WithCaptureAll<Player>()
    .WithComponentCapturer<PlayerCapturer>()
    .WithCaptureAction( x => x
        .GetTrackRecorder( scoreManager )
        .Property( "Score" )
        .Capture() );
    
var recorder = new MovieRecorder( Scene, options );
```

## Default Options

If you want to base your configuration off the default options (which captures all renderers, cameras, particles etc), you can access it through *MovieRecorderOptions.Default*.

```csharp
var options = MovieRecorderOptions.Default with { SampleRate = 60 }
    .WithFilter( x => !x.Tags.Has( "effect" ) );
```

You can customize the default options by creating one or more static methods with a *\[DefaultMovieRecorderOptions\]* attribute, like this:

```csharp
[DefaultMovieRecorderOptions]
public static MovieRecorderOptions BuildDefaultOptions( MovieRecorderOptions options )
{
	return options
		.WithFilter( x => !x.Tags.Has( "viewmodel" ) )
		.WithFilter( x => x.PrefabInstanceSource?.StartsWith( "prefabs/surface/" ) is not true );
}
```

The default options will be used when pressing the *Record* button in the Movie Maker editor, and the `movie` console command in-game.

## Sample Rate

By default, property tracks in the generated recording are resampled at 30 FPS to keep file sizes small. You can choose a different rate when creating a *MovieRecorderOptions*, or use the *with* syntax on an existing instance.

```csharp
var options = new MovieRecorderOptions( SampleRate: 60 );
```

```csharp
var options = MovieRecorderOptions.Default with { SampleRate = 60 };
```

## Filters

Adding filters lets you control which *GameObject*s are allowed to be captured. Components won't be recorded if their containing *GameObject*, or any of its ancestors, are filtered out. The first time a capture is attempted on an object, it is checked against all provided filters. If any return *false*, that object and any of its descendants will be ignored during recording.

```csharp
var options = MovieRecorderOptions.Default
    .WithFilter( x => !x.Tags.Has( "effect" ) )
    .WithFilter( x => !x.Name.StartsWith( "decal_" ) );
```

## Capture Actions

Every time *Capture* is called on a *MovieRecorder*, it goes through all the capture actions in its *MovieRecorderOptions*. These actions will then access track recorders for *GameObject*s, *Component*s, and properties, then call *Capture* on them.

```csharp
// Capture "ExampleProperty" from inside exampleComponent
var options = new MovieRecorderOptions()
    .WithCaptureAction( x => x
      .GetTrackRecorder( exampleComponent )
      .Property( "ExampleProperty" )
      .Capture() );
```

## Component Capturers

Most of the time you'll want to capture specific properties from all instances of a given component type. To simplify this, you can implement *ComponentCapturer<T>*. Whenever *Capture* is called on a component's track recorder, all component capturer instances that match that component's type will be invoked.

```csharp
class ModelRendererCapturer : ComponentCapturer<ModelRenderer>
{
	protected override void OnCapture( IMovieTrackRecorder recorder, ModelRenderer component )
	{
		recorder.Property( nameof( ModelRenderer.Model ) ).Capture();
		recorder.Property( nameof( ModelRenderer.Tint ) ).Capture();
		recorder.Property( nameof( ModelRenderer.MaterialOverride ) ).Capture();
		recorder.Property( nameof( ModelRenderer.RenderType ) ).Capture();

		if ( component.HasBodyGroups )
		{
			recorder.Property( nameof( ModelRenderer.BodyGroups ) ).Capture();
		}
	}
}
```

All capturers with public parameterless constructors will be included in *MovieRecorderOptions.Default*, or you can manually add capturers to an options instance. These capturers are invoked when *Capture* is called on a matching component's track recorder.

```csharp
var options = new MovieRecorderOptions()
    .WithComponentCapturer( new ModelRendererCapturer() )
    .WithCaptureAction( recorder =>
    {
        foreach ( var renderer in recorder.Scene.GetAllComponents<ModelRenderer>() )
        {
            // This will look for matching ComponentCapturers
            recorder.GetTrackRecorder( renderer )?.Capture();
        }
    } );
```

For convenience, *WithCaptureAll* does the same job as the above example capture action.

```csharp
var options = new MovieRecorderOptions()
    .WithComponentCapturer( new ModelRendererCapturer() )
    .WithCaptureAll<ModelRenderer>();
```
