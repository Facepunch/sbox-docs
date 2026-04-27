---
title: "Unit Tests"
icon: "🧪"
created: 2025-02-10
updated: 2026-03-04
---

# Unit Tests

If you add a  `UnitTests`  directory to your project folder, we will automatically generate a Unit Test project for you.
In order for the project to be generated you need to restart your editor if you have it open.

## Your First Test

Add a new File to the unit test project call it `MyFirstTest.cs`

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class MyFirstTest
{
	[TestMethod]
	public void Simple()
	{
		Assert.AreEqual( 4, 2 + 2 );
	}
}
```

You can run the tests using the `dotnet test` command via [CLI](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test). If your using Visual Studio you can use the Test Explorer to run your tests.

## Game Tests

If your test relies on engine functionality, for example if you want to test a component. You can initialize the engine using the following code:

```csharp
global using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class TestInit
{
	public static Sandbox.AppSystem TestAppSystem;

	[AssemblyInitialize]
	public static void AssemblyInitialize( TestContext context )
	{
		TestAppSystem = new TestAppSystem();
		TestAppSystem.Init();
	}

	[AssemblyCleanup]
	public static void AssemblyCleanup()
	{
		TestAppSystem.Shutdown();
	}
}
```

Just drop it into any file in the unit test project e.g. `InitTests.cs`

### Example Component Test

```csharp
using Sandbox;

[TestClass]
public class Camera
{
	[TestMethod]
	public void MainCamera()
	{
		var scene = new Scene();
		using var sceneScope = scene.Push();

		Assert.IsNull( scene.Camera );

		var go = scene.CreateObject();
		var cam = go.Components.Create<CameraComponent>();

		Assert.IsNotNull( scene.Camera );

		go.Destroy();
		scene.ProcessDeletes();

		Assert.IsNull( scene.Camera );
	}
}
```
