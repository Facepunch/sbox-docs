---
title: "Network Visibility"
icon: "👁️"
created: 2025-11-14
updated: 2026-07-28
---

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

:::info
The GameObject holding the component must be **enabled** for it to be asked. If it's disabled, the engine ignores it and falls back to PVS.

Networked children get checked too. A child is sent if its **root object is being sent** to that connection, **or** if its own check says it's visible. So a visible root brings its networked children with it, but a child can still be visible on its own even when the root is culled.

On the client, being active always follows the hierarchy. A child under a culled parent won't be active, even if it's still getting updates.
:::


---

## Hammer PVS Integration

If **no** component implementing `INetworkVisible` exists on the root GameObject *and* the map is a **Hammer map** with VIS compiled:

* The engine automatically falls back to **PVS (Potentially Visible Set)**.
* Visibility is determined using Hammer's visibility data, checked against where the player can see from.

This is an ideal default for static world objects on Hammer maps.

If there's no PVS data to check against - no Hammer map, or VIS wasn't compiled - everything counts as visible, so nothing gets culled even with `AlwaysTransmit` turned off. On those maps you need an `INetworkVisible` component to cull anything.


---

## What Happens When an Object Is Culled?

It depends on whether the object has ever been sent to that connection.

### Before it's first sent

If a connection has never seen the object, it **doesn't have it at all**. The object isn't spawned on that client, so it isn't there in any form - not even hidden. It spawns the first time it becomes visible to that connection.

This only matters for players who **join after the object already exists**. If an object is network spawned while you're already connected, it's created on your client straight away, whether you can see it or not - culling then just turns it off. The filtering happens in the snapshot a player gets when they join. A joining player only gets objects that are visible to them right then, plus anything with `AlwaysTransmit` on, and anything in `NetworkMode.Snapshot`.

An object is also included if one of its networked **children** is visible, because a child is never sent without its parents. So an object you'd expect to be culled can still show up on a client because something under it was visible.

### After it's been sent

Once the object has spawned for a connection, it **stays on that client for good**. Culling never destroys it - it just hides it while it's not visible.

While it's hidden, for that connection:

* Sync Var updates and Transform updates stop being sent.
* **RPCs are still delivered.** They still run, even though the object is inactive.


Hidden objects are still known to the client once they've been seen, they just stop getting updates. When they become visible again, the same object turns back on and starts getting updates - it doesn't respawn.
