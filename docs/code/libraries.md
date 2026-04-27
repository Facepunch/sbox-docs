---
title: "Libraries"
icon: "📗"
created: 2024-04-27
updated: 2025-06-15
---

# Libraries

Libraries can be used to create self-contained code and assets that you can reuse in multiple projects. These can be shared with your organisation or with the whole community.

## How it works

Libraries are stored in the `Libraries` folder in your project. Each library has its own folder, with its own `Assets`, `Code`, and `Editor` folders.

The game code, and game's editor code reference all installed libraries, so it can access any classes you define in the library. But a library cannot access the game code directly.

Libraries are not compiled assemblies. When you share your library you are sharing the source code. You are sharing everything in the library folder.

Libraries aren't downloaded and restored automatically. The expectation is that you download a library to your `<project>/Libraries/` folder, and you then commit that to your source control.

## Why it works like this

There are multiple benefits to the library system working in a way where you download it and store the code yourself.

* You fully control the code. You can look at it, edit it, delete what you don't need.
* It won't get deleted - because you have it
* If there are assets you don't need, you can delete them.
* If there are assets you want to change, you can change them.

## Library Manager

You can open the `Library Manager` tab from the `View` menu. 

This tab lets you see installed libraries, update them, remove them. 

It also lets you browse available libraries and install them.

## Publishing

Libraries can be published like any other package. Right-click the library in Library Manager and select Publish Project. The standard project visibility rules apply, so no-one outside your org will be able to see it unless you make it public.

## Limitations

Libraries cannot reference other libraries. If you are thinking of shipping a library that relies on another library - don't. It isn't supported.
