---
title: "User Permissions"
icon: "🔒"
created: 2025-01-17
updated: 2025-01-17
---

# User Permissions

On a Dedicated Server, you can specify who is an admin of your server, or customize permissions even further with claims for individual Steam accounts.

## Users

On your Dedicated Server, you can edit the `users/config.json` file to add specific permissions per user. The default config demonstrates how these are set up.

```json
[

	/* You can give a Steam account specific permissions here using their Steam Id. */
	
	{
		"SteamId": "00000000000000000",
		"Claims": [ "kick", "ban", "restart" ],
		"Name": "Example"
	},
	
	{
		"SteamId": "00000000000000000",
		"Claims": [ "kick", "ban", "restart" ],
		"Name": "Example"
	}
	
]
```

## Claims

Claims are strings which describe actions that a user can take. You can add your own custom claims, as they're just strings.

The host can check if a specific [Connection](https://sbox.game/dev/api/Sandbox.Connection) has a permission with `Connection.HasPermission( string )`. By default, the host has all permissions.

:::info
Permissions are not networked right now, so only the host can check if a connection has a specific permission.

:::
