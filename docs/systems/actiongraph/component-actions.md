---
title: "Component Actions"
sources:
  - engine/Sandbox.Engine/Systems/ActionGraphs/ActionGraphResource.cs
icon: "🧩"
created: 2024-01-08
updated: 2026-04-14
---

# Component Actions

Learn how to quickly add visual scripting to a GameObject without writing any C# code using the `Actions` component.

## Quick Working Example

1. Select a GameObject in your scene.
2. Click **Add Component** in the Inspector.
3. Under the `Actions` category, select **Actions Invoker** (or select **New ActionGraph Component...** at the very bottom).
4. In the Inspector, click on the **Empty Action** button next to `On Start` or `On Update`.
5. The ActionGraph editor will open, ready for you to build visual logic.

### The Actions Invoker Component

The `Actions Invoker` component is a general-purpose listener that exposes several common engine events to visual scripting.

![Available actions in an Actions Invoker component.](./images/available-actions-in-an-actions-invoker-component.png)

1. Add the `Actions Invoker` component to your GameObject.
2. Click **Empty Action** next to `On Update`.
3. Inside the ActionGraph editor, add a **Set WorldRotation** node.
4. Link the white execution signal from the root node to the new node.
5. You've now created a script that rotates the object every frame without touching C#!

### Trigger Actions in Colliders

Many built-in components provide their own Action properties for specific events. For example, if you add a `BoxCollider` and check the **Is Trigger** box, it exposes trigger actions.

![Trigger actions in a Collider.](./images/trigger-actions-in-a-collider.png)

1. Check **Is Trigger** on a Collider.
2. Click **Empty Action** next to `On Trigger Enter`.
3. The ActionGraph editor opens with a root node that includes an `Other` parameter (the GameObject that entered the trigger).
4. You can drag a wire from `Other` to apply logic (like dealing damage) to whoever stepped in the trigger!

## Configuration

When using the `Actions Invoker` component, these are the available events you can bind ActionGraphs to:

| Event Property | Description |
|---|---|
| `On Awake` | Fires once when the component is created, before Start. |
| `On Start` | Fires once when the component is enabled in the scene for the first time. |
| `On Update` | Fires every frame. Useful for visual effects or input reading. |
| `On Fixed Update` | Fires on a fixed physics timestep. Useful for applying physics forces. |
| `On Destroy` | Fires right before the component or GameObject is destroyed. |

## Troubleshooting

:::danger "My graph doesn't do anything when I play!"
Make sure your nodes are connected to the white signal wire coming from the Root node. If a node (like a `Set Position` node) doesn't receive a white execution signal, it will never run, even if its value wires are connected.
:::

## Related Pages
- [Intro to ActionGraphs](intro-to-actiongraphs.md)
- [Using with C#](using-with-c.md)
