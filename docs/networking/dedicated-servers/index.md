---
title: "Dedicated Servers"
icon: "settings_input_hdmi"
created: 2024-11-07
updated: 2025-04-15
---

# Dedicated Servers

# Installation

You can find information about how to install and use SteamCMD here <https://developer.valvesoftware.com/wiki/SteamCMD> to install the s&box Dedicated Server.


Once SteamCMD has been installed, you can install or update the [s&box Server](https://steamdb.info/app/1892930/info/) by running the following command from the directory you installed SteamCMD.

`./steamcmd +login anonymous +app_update 1892930 validate +quit`


:::info
You can use `-beta staging` to host a server on the staging branch, this might not be playable by everyone though.

:::

# Running the Server

Once installed, the default directory would be `steamcmd/steamapps/common/sbox dedicated server`.

## Windows

You can create a simple `.bat` file that will start your server. Here's an example, you could create a file called `Run-Server.bat` that looks like this:

```
echo off
sbox-server.exe +game facepunch.sandbox facepunch.flatgrass +hostname My Dedicated Server
```

## Linux

The server runs on .NET, so you'll need the [.NET Runtime](https://dotnet.microsoft.com/en-us/download) installed. You can create a shell script to start your server, for example `run-server.sh`:

```bash
#!/bin/bash
./sbox-server.exe +game facepunch.sandbox facepunch.flatgrass +hostname "My Dedicated Server"
```

When run, this will load the `facepunch.sandbox` game with the `facepunch.flatgrass` map and the title would be *My Dedicated Server*.


# Configuration

You can run the server with the following available command line parameters. These are just essentially a ConVar or ConCmd that is run when the server boots up.



:::info
You can pass a path to a `.sbproj` file to load a local project on a Dedicated Server. Connected clients will receive code changes and hotload them.

:::



| Switch | Arguments | Description |
|--------|-----------|-------------|
| +game  | `<packageIdent>` `[mapPackageIdent]` | The game package to load and optionally a map package. |
| +hostname  | `<name>`  | The server title that players will see. |
| +net_game_server_token  | `<token>` | **This is not required and is only available as an option once s&box is released.**Visit <https://steamcommunity.com/dev/managegameservers> to generate a token associated with your Steam Account. You can use this token to ensure your Dedicated Server always has the same Steam ID for other players to connect to it. You don't need this, but otherwise every time you load the server it will generate a new Steam ID. |
| +port  | `<port>`  | The port used to host the server on. |
| +net_query_port  | `<port>`  | The port used to query server information such as player count, current map, etc. |

# Connecting to the Server

Once your dedicated server is running, you can connect to it using the in-game console.

## Getting Connection Information

After your game has created a lobby, run the following command in the **dedicated server console**:

~~~ 
status
~~~

This will display the server’s connection details, including the **lobby ID**.

:::info
By default, s&box servers use Steam’s proxy relay system. This means connections are routed through Steam’s servers, and your dedicated server’s IP address is not exposed.
:::

## Connecting

To join the server, open the game client console and run:

~~~
connect <lobbyId>
~~~

Example:

~~~
connect 1234567890
~~~

**Important:** Do not include the `steamid:` prefix when using the lobby ID.

## Troubleshooting

- **"Not Connected" error**  
  This means a lobby has not been created in your game code yet.  
  Refer to the networking documentation to implement lobby creation.
