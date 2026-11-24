---
title: "Workshop for Dedicated Servers"
icon: "🎮"
created: 2026-11-24
updated: 2026-11-24
---

# Workshop for Dedicated Servers

Dedicated servers can use addons, games, and maps from the Steam Workshop. Items are specified with a Workshop collection or a configuration file, and the server installs new items and updates automatically each time it starts.

## Workshop IDs

Every item on the Steam Workshop has an ID. The easiest way to find it is in the item's URL:

`https://steamcommunity.com/sharedfiles/filedetails/?id={ID}`

Collections use the same URL format, so you can find a collection's ID the same way.

## Collections

The simplest way to manage Workshop content for a server is with a collection. To provide one, pass its ID on the command line:

`+host_workshop_collection 3842887665`

Once it's set up, you can add or remove items by editing the collection in Steam, with no changes needed on the server.

## Using a configuration file

Alternatively, you can list Workshop items in `config/workshop.json`.

Here's an example `config/workshop.json` that includes the Rust Props and Rust Vehicles addons:

```json
[
    3840940270,
    3828570091
]
```