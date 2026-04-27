---
title: "Component Queries"
icon: "🔍"
sources:
  - engine/Sandbox.Engine/Scene/GameObject/ComponentList.cs
  - engine/Sandbox.Engine/Scene/GameObject/GameObject.GetComponent.cs
created: 2026-04-27
updated: 2026-04-27
---

# Component Queries

Component queries allow you to find and retrieve specific components attached to a `GameObject`, its children, or its parents.

## Quick Start

```csharp
// Get a component on this GameObject
var modelRenderer = GameObject.Components.Get<ModelRenderer>();

if ( modelRenderer != null )
{
    modelRenderer.Model = Model.Load( "models/dev/box.vmdl" );
}
```

### Basic Queries

You can query components directly from a `GameObject`. These are the most common methods and return the first matching component, or an enumerable of matches. By default, queries only return enabled components.

```csharp
// Get a single component of type ModelRenderer
var renderer = GameObject.Components.Get<ModelRenderer>();

// Get all components of type ModelRenderer on this object
var allRenderers = GameObject.Components.GetAll<ModelRenderer>();
```

### Safe & Creation Queries

Sometimes you need to ensure a component exists, or you want to safely attempt to get a component without null checks cluttering your code. 

```csharp
// Gets the ModelRenderer if it exists, or creates and attaches a new one if it doesn't
var renderer = GameObject.Components.GetOrCreate<ModelRenderer>();

// Safely try to get a component, executing code only if it was found
if ( GameObject.Components.TryGet<ModelRenderer>( out var foundRenderer ) )
{
    foundRenderer.Tint = Color.Red;
}
```

### Hierarchy Queries

You can easily search for components in children (descendants) or parents (ancestors) using specific wrapper methods. This is the preferred way to do hierarchy queries.

```csharp
// Get the first ModelRenderer on this object or any of its children
var childRenderer = GameObject.Components.GetInDescendantsOrSelf<ModelRenderer>();

// Get all ModelRenderers strictly in the descendants (excluding this object)
var allChildRenderers = GameObject.Components.GetAll<ModelRenderer>( FindMode.InDescendants );

// Get the first ModelRenderer on this object or any of its parents
var parentRenderer = GameObject.Components.GetInAncestorsOrSelf<ModelRenderer>();
```

Available wrapper methods include:
- `GetInChildren<T>()` / `GetInChildrenOrSelf<T>()`: Searches immediate children.
- `GetInDescendants<T>()` / `GetInDescendantsOrSelf<T>()`: Searches all children, their children, etc.
- `GetInParent<T>()` / `GetInParentOrSelf<T>()`: Searches the immediate parent.
- `GetInAncestors<T>()` / `GetInAncestorsOrSelf<T>()`: Searches the parent, its parent, etc.

Most wrapper methods take an optional `includeDisabled` boolean parameter (defaulting to false) to include disabled components in the search.

## Advanced Configuration: FindMode

Under the hood, all queries use `GameObject.Components` and a bitmask called `FindMode`. If the provided wrapper methods don't fit your needs, you can use `GameObject.Components.Get<T>( FindMode )` or `GameObject.Components.GetAll<T>( FindMode )` for precise control over your queries.

```csharp
// Advanced: Find all enabled or disabled ModelRenderers in ancestors
var modes = FindMode.InAncestors | FindMode.Enabled | FindMode.Disabled;
var renderers = GameObject.Components.GetAll<ModelRenderer>( modes );
```

| FindMode Flag | Description |
|---|---|
| `InSelf` | Searches the current GameObject. |
| `InChildren` | Searches immediate children of the GameObject. |
| `InDescendants` | Searches all children, their children, etc. |
| `InParent` | Searches the immediate parent of the GameObject. |
| `InAncestors` | Searches the parent, its parent, etc. |
| `Enabled` | Only returns components that are currently enabled. |
| `Disabled` | Only returns components that are currently disabled. |
| `EnabledInSelfAndDescendants` | Helper flag that combines Enabled, InSelf, and InDescendants. |

## Troubleshooting

:::warning Querying Disabled Components
By default, component queries only return **enabled** components. If you need to find a disabled component, make sure to pass `true` to `includeDisabled` in the wrapper methods (e.g. `GetInDescendants<T>( true )`), or explicitly include the `FindMode.Disabled` flag if using manual masks.
:::

:::danger Performance Note
Querying components in descendants on a GameObject with a massive hierarchy can be slow if done every frame. Try to query components once in `OnStart` or `OnAwake` and cache the result in a variable.
:::

## Related Pages
* [Components Overview](../scene/components/index.md)
* [GameObject](../scene/gameobject.md)