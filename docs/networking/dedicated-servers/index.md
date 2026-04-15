---
title: "Dedicated Servers"
icon: "settings_input_hdmi"
created: 2024-11-07
updated: 2025-04-15
---

# Dedicated Servers

# Installation

You can find information about how to install and use SteamCMD here <https://developer.valvesoftware.com/wiki/SteamCMD> to install the s&box Dedicated Server.


Once SteamCMD has been installed, you can install or update the [s&box Server](https://steamdb.info/app/1892930/info/) by running the following command in Windows Terminal from the directory you installed SteamCMD.

`./steamcmd +login anonymous +app_update 1892930 validate +quit`


:::info
You can use `-beta staging` to host a server on the staging branch, this might not be playable by everyone though.

:::

# Running the Server

Once installed, the default directory would be `steamcmd/steamapps/common/Dedicated Server` and you can create a simple .bat file there that will start your server. Here's an example, you could create a file called `Run-Server.bat` that looks like this:


```csharp
echo off
sbox-server.exe +game facepunch.walker garry.scenemap +hostname My Dedicated Server
```


When run, this will load the `facepunch.walker` game with the `garry.scenemap` map and the title would be *My Dedicated Server*.


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
