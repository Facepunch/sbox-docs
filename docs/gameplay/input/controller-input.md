---
title: "Controller Input"
icon: "🎮"
created: 2025-02-25
updated: 2026-09-03
---

# Controller Input

Some Input methods are specific to Controllers/Gamepads, and are useful to know for making sure your player experience caters to those who don't play on a Keyboard + Mouse.

To see if the player is using a Controller, you can check `Input.UsingController`

# Analog Inputs

You can get direct analog inputs from the Left Stick, Right Stick, Left Trigger or Right Trigger. Keep in mind that not all Controllers have analog triggers.

```csharp
float moveX = Input.GetAnalog( InputAnalog.LeftStickX );
float aimAmount = Input.GetAnalog( InputAnalog.LeftTrigger );
```

There is also `Input.AnalogMove` and `Input.AnalogLook` which are Vector3 and Angles respectively that automatically map to both the Keyboard + Mouse and the Left and Right Stick.

# Haptics/Rumble

You can directly set the vibration intensity of each motor (and how long the vibration should last)

```csharp
float leftMotor = 0.5f; // 50% intensity
float rightMotor = 0.7f; // 70% intensity
float leftTrigger = 0.2f; // 20% intensity
float rightTrigger = 0f; // No vibration
int duration = 1000; // 1000 milliseconds == 1 second

Input.TriggerHaptics( leftMotor, rightMotor, leftTrigger, rightTrigger, duration );
```

Or you can use one of a few presets, specifying the length, frequency and intensity

```csharp
// These are optional inputs
float lengthScale = 1.2f; // 1.2x longer vibration time
float frequencyScale = 2f; // 2x vibration frequency
float amplitudeScale = 0.5f; // 0.5x as intense of a rumble

Input.TriggerHaptics( HapticEffect.HardImpact, lengthScale, frequencyScale, amplitudeScale );
```

You can also force all haptics to stop using `Input.StopAllHaptics()`

# Motion Controls

If the Controller has a gyroscope or an accelerometer, then you can get motion data, otherwise the values will be zero.

```csharp
InputMotionData motionData = Input.MotionData;
Vector3 acceleration = motionData.Accelerometer; // Accelerometer
Angles gyroscope = motionData.Gyroscope; // Gyroscope
// Process the data as needed...
```

# Local Multiplayer

You can see how many Controllers are currently connected using `Input.ControllerCount`

With that count you could create a Player GameObject for each index, and then query their Inputs as normal

```csharp
int playerIndex = 0; // The index of the controller this player is using

using( Input.PlayerScope( playerIndex ) )
{
  // Any input handling within this code block will only query Controller index 0
  if( Input.Pressed( "Jump" ) )
  {
    // Make Player 1 jump (or whichever index)
  }
}
```
