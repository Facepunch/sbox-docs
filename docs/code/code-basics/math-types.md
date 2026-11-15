---
title: "Math Types"
icon: "📐"
created: 2025-06-16
updated: 2026-11-15
---

# Math Types

## Vectors

A **Vector** is just a set of numbers that describe a position or a direction in space. 


* **Vector2** - for flat 2D stuff (like a screen): x and y.
* [**Vector3**](https://sbox.game/api/Vector3) - for 3D stuff: x, y, z.
* **Vector4** - same as Vector3 but with a bonus W value. Used commonly for shaders, or margins, or borders, or corners.
* **Vector2Int + Vector3Int** - like normal, but with whole numbers only.


```csharp
// Make a new vector
var a = new Vector3(1, 2, 3);
var b = new Vector3(4, 5, 6);

// Get or set the values
float x = a.x;
a.z = 10;

// You can add and subtract them
var sum = a + b;
var difference = b - a;

// You can scale them (make them bigger or smaller)
var twiceAsBig = a * 2;
var halfAsBig = a / 2;

// Get how long the vector is (like the length of the arrow)
float length = a.Length;

// Make it point in the same direction but only 1 unit long
Vector3 normal = a.Normal;

// How far apart two vectors are
float distance = a.Distance(b);

// Dot product tells how similar two directions are
float dot = Vector3.Dot(a, b);

// Cross product gives you a new direction that's 90° to both
Vector3 cross = Vector3.Cross(a, b);

// Lerp moves smoothly from a to b (0.5 is halfway)
Vector3 halfway = Vector3.Lerp(a, b, 0.5f);
```


## Rotation & Angles

A [**Rotation**](https://sbox.game/api/Rotation) tells you which way something is facing in 3D. It's technically a Quaternion.

There's also [**Angles**](https://sbox.game/api/Angles), which uses pitch, yaw, and roll. It's easier to write by hand, but it can suffer from gimbal lock. That's why we mostly use Rotation in code - it avoids those problems and is more reliable.


```csharp
// Make some angles (easy to understand)
Angles ang = new Angles(30, 90, 0);  // pitch, yaw, roll

// Convert to Rotation
Rotation rot = ang.ToRotation();

// Convert back to Angles
Angles ang2 = rot.Angles();

// Get direction vectors from a Rotation
Vector3 fwd = rot.Forward;
Vector3 up = rot.Up;
Vector3 right = rot.Right;

// Make a Rotation that looks in a direction
Rotation look = Rotation.LookAt(new Vector3(1, 0, 0), Vector3.Up);

// Create basic axis rotations
Rotation ry = Rotation.FromYaw(90);    // turn left 90°
Rotation rx = Rotation.FromPitch(45);  // look down 45°
Rotation combined = ry * rx;           // do both

// Smoothly rotate between two rotations
Rotation start = Rotation.FromYaw(0);
Rotation end = Rotation.FromYaw(180);
Rotation halfway = Rotation.Slerp(start, end, 0.5f);

// Rotate a direction vector
Vector3 forward = Vector3.Forward;
Vector3 rotated = rot * forward;

// Invert a rotation
Rotation inverse = rot.Inverse;
```


## Transform

A [**Transform**](https://sbox.game/api/Transform) holds three things:

* a **Position (Vector3)**,
* a **Rotation (Rotation)**,
* and a **Scale (Vector3)**

In this way, it can fully represent the "transform" of a GameObject. On a GameObject it's available in two forms.. 

* `GameObject.WorldTransform` - position in the world
* `GameObject.LocalTransform` - position relative to its parent


```csharp
// Create a basic Transform at origin, facing forward
Transform t = Transform.Zero;

// Or with custom position and rotation
Vector3 pos = new Vector3(10, 0, 0);
Rotation rot = Rotation.FromYaw(90);
Transform custom = new Transform(pos, rot, 1f);  // scale is 1

// Access individual parts
Vector3 p = custom.Position;
Rotation r = custom.Rotation;
Vector3 scale = custom.Scale;

// Get direction vectors from the transform
Vector3 forward = custom.Forward;
Vector3 up = custom.Up;
Vector3 right = custom.Right;

// Move a point from local space to world space
Vector3 local = new Vector3(1, 0, 0);
Vector3 world = custom.PointToWorld(local);

// Move a point from world space to local space
Vector3 backToLocal = custom.PointToLocal(world);

// You can also build a Matrix from the Transform's parts
Matrix mat = Matrix.CreateScale(custom.Scale)
	* Matrix.CreateRotation(custom.Rotation)
	* Matrix.CreateTranslation(custom.Position);

// And use that Matrix to transform a point
Vector3 transformed = mat.Transform(local);
```
