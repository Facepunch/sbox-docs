---
title: "Player Controller"
icon: "🏃‍♀️"
sources:
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.cs
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.Input.cs
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.Camera.cs
  - engine/Sandbox.Engine/Scene/Components/Game/PlayerController/PlayerController.Events.cs
updated: 2026-04-25
created: 2026-04-27
---

# Player Controller

The `PlayerController` component is a ready-to-use first and third-person character controller that handles movement, physics collisions, and input natively.

## Quick Working Example

You can listen to events from the `PlayerController` by implementing `PlayerController.IEvents` on another Component attached to the same GameObject.

```csharp
public class MyPlayerLogic : Component, PlayerController.IEvents
{
    public void OnJumped()
    {
        Log.Info( "The player just jumped!" );
    }

    public void OnLanded( float distance, Vector3 impactVelocity )
    {
        Log.Info( $"The player landed after falling {distance} units." );
    }

    public void OnEyeAngles( ref Angles angles )
    {
        // You can intercept and modify eye angles (e.g. for recoil)
    }
}
```

### Customizing Input
By default, the `PlayerController` uses the built-in `Input.AnalogMove` and `Input.AnalogLook`. If you want to drive the controller using AI or your own custom input logic, disable the built-in input controls (`UseInputControls = false`) and set the `WishVelocity` and `EyeAngles` manually in `OnFixedUpdate`.

```csharp
public class AIController : Component
{
    [RequireComponent] PlayerController Player { get; set; }

    protected override void OnStart()
    {
        // Disable built-in input handling for this controller
        Player.UseInputControls = false;
    }

    protected override void OnFixedUpdate()
    {
        // Move forward constantly using world position changes
        Player.WishVelocity = new Vector3( 150f, 0, 0 );
        
        // Jump occasionally
        if ( Time.Tick % 100 == 0 && Player.IsOnGround )
        {
            Player.Jump( Vector3.Up * Player.JumpSpeed );
        }
    }
}
```

### Interacting with Objects (Pressing)
The `PlayerController` has a built-in mechanism for interacting with `IPressable` components. If `EnablePressing` is true, the player can look at an object and press the assigned `UseButton` (default: "use").

```csharp
public class Door : Component, Component.IPressable
{
    public bool CanPress( IPressable.Event e ) => true;

    public bool Press( IPressable.Event e )
    {
        Log.Info( "The player opened the door!" );
        return true; // Return true to acknowledge the press; Release will be called later.
    }

    // Unused overrides
    public void Release( IPressable.Event e ) {}
    public void Look( IPressable.Event e ) {}
    public void Blur( IPressable.Event e ) {}
}
```

## Configuration

### Input Controls
These properties configure the default player input behaviors when `UseInputControls` is active.

| Property | Description |
|:---|:---|
| `UseInputControls` | Enable to use the default `AnalogMove` and `AnalogLook` controls. |
| `WalkSpeed` | The movement speed when walking (default: 110). |
| `RunSpeed` | The movement speed when running (default: 320). |
| `DuckedSpeed` | The movement speed when ducking (default: 70). |
| `JumpSpeed` | The upward velocity applied when jumping (default: 300). |
| `AccelerationTime` | Amount of seconds it takes to reach requested speed when accelerating (default: 0). |
| `DeaccelerationTime` | Amount of seconds it takes to reach requested speed when decelerating (default: 0). |
| `AltMoveButton` | The action button used to run (default: "run"). |
| `RunByDefault` | If true, the player will run by default, and `AltMoveButton` will switch to walk. |
| `UseLookControls` | If true, the camera pitch/yaw will update based on `AnalogLook`. |
| `LookSensitivity` | Modifies the eye angle sensitivity (default: 1). |
| `PitchClamp` | Limits how far up and down the player can look (default: 90). |

### Camera
These properties configure the built-in camera behaviors.

| Property | Description |
|:---|:---|
| `UseCameraControls` | Built-in camera controls. Remove this to control the Camera yourself. |
| `EyeDistanceFromTop` | Offset of the eyes from the top of the body collider (default: 8). |
| `ThirdPerson` | If true, the camera will be positioned in third person (default: true). |
| `HideBodyInFirstPerson` | If true, hides the player's body in first person (default: true). |
| `UseFovFromPreferences` | Overrides the field of view with the user's preference (default: true). |
| `CameraOffset` | The local offset for the third-person camera (default: 256, 0, 12). |
| `ToggleCameraModeButton` | The action button used to toggle between first and third person (default: "view"). |

### Pressing
| Property | Description |
|:---|:---|
| `EnablePressing` | Allows the player to interact with objects by pressing a use button. |
| `UseButton` | The action button used to interact with objects (default: "use"). |
| `ReachLength` | How far from the eye the player can reach to use things (default: 130). |

### Body
| Property | Description |
|:---|:---|
| `BodyRadius` | The radius of the player's bounding collider (default: 16). |
| `BodyHeight` | The height of the character when standing (default: 72). |
| `DuckedHeight` | The height of the character when ducking (default: 36). |
| `BodyMass` | The total physical mass of the player (default: 500). |

### Physics
| Property | Description |
|:---|:---|
| `BrakePower` | Extra friction applied when on the ground to slow down when needed. |
| `AirFriction` | How much friction is added when airborne without a wish velocity. |

## Troubleshooting

:::warning "My character isn't moving when I press WASD!"
Ensure that **UseInputControls** is enabled on the `PlayerController`. If you disabled it to handle logic manually, you must manually update its properties (like `WishVelocity` and `EyeAngles`) in `OnFixedUpdate`.
:::

:::warning "I need to configure my camera after the player spawns."
If you need to tweak the camera component linked to the player controller, implement `PlayerController.IEvents` and use the `PostCameraSetup( CameraComponent cam )` method.
:::

:::danger "The player falls through the floor!"
Verify that the `PlayerController` has instantiated its internal colliders correctly, and that the ground object has a `Collider` component attached. Ensure `BodyCollisionTags` includes the collision tags appropriate for the ground layers.
:::

## Related Pages
* [Physics / Rigidbody](../../../systems/physics/rigidbody.md)
* [Character Controller](charactercontroller.md)
* [Input System](../../../systems/input/index.md)