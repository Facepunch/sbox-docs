---
title: "Serverside Code"
icon: "👾"
created: 2025-07-09
updated: 2025-07-09
---

# Serverside Code

:::info
Serverside Code only works when running local projects - please read that page first.

:::

# What is it?

When compiling a project, if we're a dedicated server, we can run code in `if #SERVER` blocks.

```csharp
protected override void OnUpdate()
{
#if SERVER
  Log.Info( $"This is a server update!" );
#else
  Log.Info( $"This is a client update!" );
#endif
}
```

If we're running on a dedicated server, we'll get `This is a server update!` spamming our console.

This is especially useful for omitting potentitally sensitive code from client builds, like contacting an API server, or a database.

# **.Server.cs files**

Say you have a `GameManager.cs` file, which has a partial class, you can accompany it with a `GameManager.Server.cs` file which is safely omitted from client builds and wraps the whole file in an `'#if SERVER` block - it's a nice bit of sugar that saves you having a bunch of preprocessor blocks in your code.

# What if I publish my game?

All serverside code is stripped out of published games by default - so there's no worries there.
