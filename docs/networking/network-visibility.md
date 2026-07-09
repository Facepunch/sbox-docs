---
title: "Network Visibility"
icon: "👁️"
created: 2025-11-14
updated: 2026-07-09
---

# Network Visibility

# Network Visibility & Culling

Network Visibility controls **whether a networked object should be visible for a specific player (Connection)**. Visibility determines whether the object receives ongoing network updates — such as Sync Vars and Transform updates — for that client.

By default, **all networked objects always transmit to all Connections**, unless you explicitly disable this behaviour.


---

## Always Transmit

Every networked object has flag called **Always Transmit**.

* **Default:** `AlwaysTransmit = true`
* When `true`, the object **never** gets culled and is visible to every player.

This is the simple default for beginners, but for larger or more complex games, disabling Always Transmit can enable performance benefits by culling objects that the player should not receive updates for.


---

## INetworkVisible Interface

You can take control of visibility by attaching a `Component` to the **root GameObject** of a networked object that implements `Component.INetworkVisible`.

Only the **owner** of a networked object decides visibility for each connection.

```csharp

public class MyVisibilityComponent : Component, INetworkVisible
{
    public bool IsVisibleToConnection( Connection connection, in BBox worldBounds )
    {
        // Your visibility logic here…
        return true;
    }
}
```

:::info
Objects aren't culled the instant `IsVisibleToConnection` returns false. There's a **2 second cull delay** - a connection has to be continuously invisible for that long before it actually stops receiving updates. If visibility flips back to true at any point during those 2 seconds, the timer resets and the object is never culled.

This avoids needlessly toggling network updates on and off for connections that are only briefly or intermittently out of view. Becoming visible again isn't delayed - a culled connection starts receiving updates again as soon as `IsVisibleToConnection` returns true.
:::

### IsVisibleToConnection Parameters

| Parameter | Description |
|-----------|-------------|
| `Connection connection` | The target player being tested. |
| `BBox worldBounds` | The object's world-space bounding box. Helpful for distance or frustum checks. |

Return **true** if the object should be visible to that connection; **false** if it should be culled.

To enable this behaviour, disable `AlwaysTransmit` on the root network object.


---

## Hammer PVS Integration

If **no** component implementing `INetworkVisible` exists on the root GameObject *and* the map is a **Hammer map** with VIS compiled:

* The engine automatically falls back to **PVS (Potentially Visible Set)**.
* Visibility is determined using Hammer's visibility data.

This is an ideal default for static world objects on Hammer maps.


---

## What Happens When an Object Is Culled?

When the owner decides an object is **not visible** to a connection:

### The following *stop being sent*:

* Sync Var updates
* Transform updates

### The following *still occur*:

* The object **still spawns** for the client.
* The object becomes **Disabled** locally for that client while hidden.
* **RPCs are still delivered**.

Invisible objects remain known to the client, just not updated.
