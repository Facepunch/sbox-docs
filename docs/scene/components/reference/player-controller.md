---
title: "Player Controller"
icon: "🏃‍♀️"
created: 2024-10-30
updated: 2026-09-03
---

# Player Controller

The PlayerController component is a first and third person player controller. 

It is made to be simple and easy to use. You should be able to add it to a scene and be running around within seconds - with no programming.

## Physics Based

In its core it's a special RigidBody. It exists in the physics system.  It has all the same properties as a regular RigidBody - like velocity and mass, except we do some extras to make it more controllable.

## Input

It has input built in. The owner of the GameObject can control it using the mouse, keyboard or game controller. 

If you're not using the built in Input Controller, then you can control it in a `OnFixedUpdate` by changing its `EyeAngles` and `WishVelocity`.


:::info
The input feature is optional. Right click on the input tab to disable it.

:::

## Camera

It has a camera controller built in. This is a third person and first person camera, which can be toggled by configuring a button to toggle it.


:::info
The camera feature is optional. Right click on the camera tab to disable it.

:::


## Animator

It has a built in animator. This will animate anything with a Citizen compatible AnimGraph.


:::info
The animator feature is optional. Right click on the animator tab to disable it.

:::


## Events

 When using the built in components the PlayerController outputs events to its GameObject. You can listen to these by implementing `PlayerController.IEvents` on a sibling Component. These will allow you to alter the eye angle input, adjust the camera after it's been set up and react to events such as landing.


```csharp
	/// <summary>
	/// Events from the PlayerController
	/// </summary>
	public interface IEvents : ISceneEvent<IEvents>
	{
		/// <summary>
		/// Our eye angles are changing. Allows you to change the sensitivity, or stomp all together.
		/// </summary>
		void OnEyeAngles( ref Angles angles ) { }

		/// <summary>
		/// Called after we've set the camera up
		/// </summary>
		[Obsolete( "Implement ICameraModifier instead - it runs in the camera's ordered modifier chain, after the player's view (order 0)." )]
		void PostCameraSetup( CameraComponent cam ) { }

		/// <summary>
		/// The player has just jumped
		/// </summary>
		void OnJumped() { }

		/// <summary>
		/// The player has landed on the ground, after falling this distance.
		/// </summary>
		void OnLanded( float distance, Vector3 impactVelocity ) { }

		/// <summary>
		/// Used by the Using system to find components we can interact with.
		/// By default we can only interact with IPressable components.
		/// Return a component if we can use it, or else return null.
		/// </summary>
		Component GetUsableComponent( GameObject go ) { return default; }

		/// <summary>
		/// We have started using something (use was pressed)
		/// </summary>
		void StartPressing( Component target ) { }

		/// <summary>
		/// We have stopped using something
		/// </summary>
		void StopPressing( Component target ) { }

		/// <summary>
		/// We pressed USE but it did nothing
		/// </summary>
		void FailPressing() { }

		/// <summary>
		/// We have a chance to do something before input is processed
		/// </summary>
		void PreInput() { }

	}
```
