---
title: "Networking & Multiplayer"
icon: "🧑‍🤝‍🧑"
created: 2023-11-24
updated: 2026-09-04
---

# Networking & Multiplayer

Networking is built right into s&box, so turning a single-player scene into a shared multiplayer experience takes surprisingly little code. GameObjects can be easily replicated to other clients, RPCs can be sent to the server or peer-to-peer and there is full dedicated server support.

:::warning
The networking system in s&box is purposefully simple and easy. Our initial aim isn't to provide a bullet proof server-authoritative networking system. Our aim is to provide a system that is really easy to use and understand.
:::

## Ways to Use It

- **Lobbies** - peer-to-peer games where one player hosts a lobby and others join
- **Dedicated Servers** - run persistent, server-authoritative, headless servers for your game
- **Custom Networking** - talk to external services over HTTP and WebSockets

## How It Works

The quickest way to get going is the **Network Helper** component - drop it into your scene and you have a working multiplayer setup, with players spawning and connecting out of the box.

Under the hood, everything revolves around **networked GameObjects**. Any GameObject can be made networked by calling `NetworkSpawn()`, after which its `[Sync]` properties replicate to everyone and its RPCs can be invoked remotely. From there, **ownership**, **visibility**, and **permissions** give you fine-grained control over who can do what, while **network events** let your components react to players joining and leaving.

## Pages

### [Network Helper](/networking/network-helper.md)

The easiest way to get a multiplayer game running - a ready-made component that handles spawning, connecting, and common setup. Start here, then customise it or write your own.

### [Networked Objects](/networking/networked-objects.md)

Make any GameObject networked with `NetworkSpawn()` so it exists for every connected player.

### [Ownership](/networking/ownership.md)

Assign and check which connection owns a networked object.

### [Sync Properties](/networking/sync-properties.md)

Replicate a property's latest value to other players whenever it changes using the `[Sync]` attribute.

### [RPC Messages](/networking/rpc-messages.md)

Call component methods remotely across the network.

### [Network Events](/networking/network-events.md)

React to players joining, leaving, and objects spawning in the scene.

### [Network Visibility](/networking/network-visibility.md)

Control whether a networked object is sent to a specific player.

### [Host Migration](/networking/host-migration.md)

When the host leaves, another player takes over from a snapshot. What survives, what doesn't, and how to keep your game running through it.

### [Custom Snapshot Data](/networking/custom-snapshot-data.md)

Attach extra data to the snapshot new players receive when they join a server.

### [Connection Permissions](/networking/connection-permissions.md)

Let the host adjust what an individual connection is allowed to do.

### [Testing Multiplayer](/networking/testing-multiplayer.md)

Tools and tricks for testing your game without a second player on hand.

### [Dedicated Servers](/networking/dedicated-servers/)

Install, configure, and run standalone s&box servers.

### [Platform](/platform/index.md)

Text chat, voice chat and your game's own menu, built in.

### [Http Requests](/networking/http-requests.md)

Make asynchronous GET/POST/DELETE requests with built-in JSON support.

### [WebSockets](/networking/websockets.md)

Connect to external servers for custom networking or persistent data.
