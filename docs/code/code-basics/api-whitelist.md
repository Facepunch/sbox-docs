---
title: "Api Whitelist"
icon: "🛡️"
created: 2025-07-17
updated: 2025-07-18
---

# Api Whitelist

We prevent access to classes and functions that could be used maliciously by whitelisting what can be used.

When playing s&box, any code that doesn't pass these checks **will not** be loaded. In the editor, the compiler will generate a `SB1000 Whitelist Error: ``*'x' is not allowed when whitelist is enabled*`.

Editor code, including libraries, doesn't have to follow these restrictions. If you are developing a standalone game, you can opt out, but you won't be able to publish to the platform while the whitelist is disabled. 


## Reporting a False Positive

If there's an API you need access to that isn't in the whitelist and you believe it's harmless, please report it on our [issue tracker](https://github.com/Facepunch/sbox-public/issues) with your reasoning, and we'll review it. Please include the symbol as it appears in the error.


## Reporting a Vulnerability

As bugs in this system represent serious security issues, please report discoveries as described [here](https://facepunch.com/security).\n***Do not report them publicly***.


## Cheat Sheet

For many standard .NET APIs that are blocked, we provide similar functionality via a safe sandboxed API of our own.

Here are some of the common ones to help new devs. Check out our full API reference [here](https://sbox.game/api/).

| ❌ Not allowed | ✅ Allowed |
|---------------|-----------|
| `Console.Log` | Use `Log.Info` as a drop-in replacement. |
| `System.IO*`  | Most standard .NET IO isn't allowed, but you can use our [Filesystem](https://sbox.game/dev/doc/systems/file-system/) API for storage of user data. |
