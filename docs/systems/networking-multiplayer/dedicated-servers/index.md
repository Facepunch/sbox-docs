---
title: "Dedicated Servers"
icon: "🔌"
created: 2024-11-07
updated: 2026-04-25
sources:
  - engine/Sandbox.Engine/Systems/Networking/System/Channel/Connection.cs
---

# Dedicated Servers

Dedicated servers allow you to run headless, always-online instances of your multiplayer games that players can join at any time. Both Windows and Linux are supported.

## Quick Working Example

You can start a dedicated server from the command line, pointing it to a specific game package and map.

```bash
# Start a dedicated server running 'sandbox' on the 'flatgrass' map
sbox-server.exe +game facepunch.sandbox facepunch.flatgrass +hostname "My Dedicated Server" +maxplayers 16
```

## Installation

You must install the s&box Dedicated Server using SteamCMD. 

1. Install [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD).
2. Open your terminal in the SteamCMD directory and run:

```bash
./steamcmd +login anonymous +app_update 1892930 validate +quit
```

This will download the server files to `steamcmd/steamapps/common/sbox dedicated server`.

:::tip
If you want to host a server on the staging branch (to test upcoming engine features), append `-beta staging` to the `app_update` command.
:::

## Running the Server

### Windows

Create a `.bat` file (e.g. `Run-Server.bat`) in the install directory:

```
echo off
sbox-server.exe +game facepunch.sandbox facepunch.flatgrass +hostname "My Dedicated Server"
```

Double-click it to launch the server.

### Linux

The server runs on .NET, so you'll need the [.NET Runtime](https://dotnet.microsoft.com/en-us/download) installed. Create a shell script (e.g. `run-server.sh`) in the install directory:

```bash
#!/bin/bash
./sbox-server.exe +game facepunch.sandbox facepunch.flatgrass +hostname "My Dedicated Server"
```

Make it executable with `chmod +x run-server.sh` and run it with `./run-server.sh`.

When run, this will load the `facepunch.sandbox` game with the `facepunch.flatgrass` map and the title would be *My Dedicated Server*.

## Configuration

You configure the server by passing command line arguments when running `sbox-server.exe`. These arguments act as Console Variables (ConVars) that are executed when the server boots.

| Argument | Example | Description |
|---|---|---|
| `+game` | `+game facepunch.sandbox` | The package Ident of the game you want to host. |
| `[map]` | `+game facepunch.sandbox facepunch.flatgrass` | Optional map package Ident, placed immediately after the game ident. |
| `+hostname` | `+hostname "My Server"` | The name that players will see in the server browser. |
| `+maxplayers`| `+maxplayers 32` | The maximum number of players allowed to connect. |
| `+port` | `+port 27015` | The UDP port the server hosts on. |
| `+net_query_port` | `+net_query_port 27016` | The port used to query server information (player count, current map, etc.). |

### Running a Local Development Server

You don't need to upload your game to the asset party to test it on a dedicated server. You can pass the path to your local project's `.sbproj` file directly.

```bash
sbox-server.exe +game "C:\Projects\MyGame\mygame.sbproj"
```

When you do this, any code changes you save in your IDE will be hotloaded onto the dedicated server, and connected clients will also receive the updated code automatically.

## Troubleshooting

:::warning Port Forwarding
If your server is running but players cannot connect over the internet, ensure you have forwarded the necessary ports on your router. s&box servers typically require port **27015** (UDP and TCP) to be open.
:::

:::warning Authority Differences
If your game works in the editor (where the host and client are the same machine) but fails on a dedicated server, you likely have an Authority issue. Ensure you are not trying to read client-only inputs (like `Input.Pressed`) on the server without checking `IsProxy`, and that networked variables are being set by the **Host** or the **Owner**.
:::

## Related Pages
* [Networking Basics](../index.md)
