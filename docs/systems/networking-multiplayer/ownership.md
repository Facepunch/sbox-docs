---
title: "Ownership"
icon: "🛍️"
created: 2023-11-24
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Scene/Networking/NetworkObject.cs
  - engine/Sandbox.Engine/Scene/Networking/NetworkObject.DataTable.cs
---

# Ownership

> **New to networking?** Read **[Networking & Multiplayer](index.md)** first — that's the overview. This is the reference for one piece (network ownership); the systems landing covers the full picture.

Networked GameObjects can be owned by a specific client connection, allowing them to control its physical movement and synced properties.

## Quick Working Example

```csharp
public sealed class PickupComponent : Component
{
    [Property] public GameObject ObjectToPickup { get; set; }

    public void TryPickup()
    {
        // 1. Take ownership of the object, so we are now simulating it
        ObjectToPickup.Network.TakeOwnership();

        // 2. Parent it to our player so we carry it
        ObjectToPickup.SetParent( GameObject );
    }

    public void DropObject()
    {
        if ( ObjectToPickup.IsValid() )
        {
            // 3. Unparent it and drop ownership back to the host
            ObjectToPickup.SetParent( null );
            ObjectToPickup.Network.DropOwnership();
        }
    }
}
```

### Checking if you are in control (`IsProxy`)

The most common pattern when writing networked components is deciding if your local machine should run the logic. If you are not the owner of the object, it is considered a "proxy" and you should usually ignore input or physics logic.

```csharp
protected override void OnUpdate()
{
    // If we are a proxy, someone else is controlling this. Do nothing.
    if ( IsProxy ) return;
    
    // We own this! Read local input and move the object.
    if ( Input.Pressed( "use" ) )
    {
        // Handle input
    }
}
```

### Allowing other players to take ownership

By default, only the Host can change the owner of an object. If you want any player to be able to take ownership (e.g., picking up a shared ball), you must change its `OwnerTransfer` rule.

```csharp
protected override void OnStart()
{
    if ( !IsProxy )
    {
        // Make it so anyone can take ownership of this object
        GameObject.Network.SetOwnerTransfer( OwnerTransfer.Takeover );
    }
}
```

## Configuration

### Owner Transfer Modes

You can control who is allowed to change the owner of an object using `GameObject.Network.SetOwnerTransfer()`.

| Mode | Behaviour |
|------|-----------|
| `OwnerTransfer.Fixed` | *(Default)* Only the Host can assign or change the owner. |
| `OwnerTransfer.Takeover` | Anyone can freely take ownership at any time. |
| `OwnerTransfer.Request` | Clients must send a request to the Host to change the owner. |

### Disconnection & Orphaned Mode

When a player disconnects, what happens to the objects they owned? You can configure this using `GameObject.Network.SetOrphanedMode()`. Note that only the current owner (or Host) can change this setting.

| Mode | Behaviour |
|------|-----------|
| `NetworkOrphaned.Destroy` | *(Default)* The object is immediately destroyed for everyone. |
| `NetworkOrphaned.Host` | The Host automatically takes over ownership. |
| `NetworkOrphaned.Random` | A random remaining client is assigned ownership. |
| `NetworkOrphaned.ClearOwner` | The object remains, but ownership is cleared (the Host simulates it, but it is technically unowned). |

## Troubleshooting

:::danger "My object is stuttering or bouncing back!"
If two players try to update the position of an object at the same time, or if a client tries to move an object they don't own, the Host will constantly correct their position, causing rubber-banding. **Always** check `if ( IsProxy ) return;` before moving networked objects.
:::

## Related Pages
* [Networked Objects](networked-objects.md)
* [Sync Properties](sync-properties.md)
