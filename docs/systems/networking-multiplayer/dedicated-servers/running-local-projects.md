---
title: "Running Local Projects"
icon: "📂"
created: 2025-07-09
updated: 2025-07-09
---

# Running Local Projects

When running a dedicated server - you have the option of running an entirely local project as the game. I.e:

`+game c:/git/sbox-mygame/.sbproj`

Assets will still be fetched from the cloud version of the game - and stuff that is missing will be sent directly from the server to clients, including code. 

# What about Serverside Code?

This is the **only opportunity** for running serverside code which can be safely hidden from clients. When publishing a game, we'll strip out serverside code - this gives you the control, via your own servers.
