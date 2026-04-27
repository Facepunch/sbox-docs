---
title: "Razor Components"
icon: "📝"
created: 2024-09-24
updated: 2026-04-12
sources:
  - engine/Sandbox.Engine/Systems/UI/Panel/Panel.cs
  - engine/Sandbox.Engine/Scene/Components/UI/PanelComponent.cs
---

# Razor Components

Razor Components allow you to create modular, reusable UI pieces (like buttons, cards, or lists) that you can easily embed into other Razor panels.

## Quick Working Example

Define a reusable UI component in a `.razor` file, for example `InfoCard.razor`:

```csharp
<root>
    <div class="header">@Header</div>
    <div class="body">@Body</div>
</root>

@code
{
    // Define fragments to let parent panels pass in content
    public RenderFragment Header { get; set; }
    public RenderFragment Body { get; set; }
}
```

Then, you can use this component inside another Razor file just like an HTML tag:

```csharp
<root>
    <InfoCard>
        <Header>Player Stats</Header>
        <Body>
            <div class="title">Level: 42</div>
        </Body>
    </InfoCard>
</root>
```

### Elements Inside Panels (ChildContent)

If you just want to pass a block of UI into a component without naming specific fragments (like `Header` or `Body`), you can use the default `ChildContent` fragment. Any elements placed directly inside the component tag will be mapped to `ChildContent`.

```csharp
// Inside MyButton.razor
<root>
    <div class="btn-background">
        @ChildContent
    </div>
</root>

@code {
    public RenderFragment ChildContent { get; set; }
}
```

```csharp
// Usage
<MyButton>
    <label>Click Me!</label>
</MyButton>
```

### Components With Data Arguments

Sometimes a component needs to render a list of items and needs the parent to define *how* each item should look. You can use `RenderFragment<T>` to pass an argument from the component back to the parent's template.

```csharp
// PlayerList.razor
@using Sandbox;

<root>
    @foreach ( var player in PlayerList )
    {
        <div class="row">
            @PlayerRow( player )
        </div>
    }
</root>

@code
{
    public List<Player> PlayerList { get; set; }
    
    // A fragment that takes a Player object as an argument
    public RenderFragment<Player> PlayerRow { get; set; }
}
```

When using this component, you use the `Context` attribute to name the variable you'll receive.

```csharp
<root>
    <PlayerList PlayerList=@MyPlayerList>
        <PlayerRow Context="item">
            <!-- You must cast the context variable to the correct type -->
            @if (item is not Player player) return;

            <div class="row">@player.Name</div>
            <div class="row">@player.Health</div>
        </PlayerRow>
    </PlayerList>
</root>

@code 
{
    public List<Player> MyPlayerList { get; set; } = new();
}
```

### Generic Components

You can make a generic Panel class by using `@typeparam` to define the type parameter. This is useful for building generic dropdowns or lists.

```csharp
// MyGenericList.razor
@typeparam T

<root>
    This is a list of type @typeof(T).Name
</root>

@code
{
    public List<T> Items { get; set; } = new();
}
```

When using the generic component in Razor, you must specify the type using the `T` attribute:

```csharp
<root>
    <MyGenericList T="string"></MyGenericList>
</root>
```

## Troubleshooting

:::danger Context Type Casting
When using `RenderFragment<T>` and the `Context` attribute, the type of the context variable defaults to `object`. You must cast it to your expected type (e.g., `if (context is not MyType item) return;`) before accessing its properties.
:::

:::warning "Component tag is not recognized!"
Ensure that the namespace of your reusable component is either in the same namespace as the parent, or you have included the correct `@using` directive at the top of your parent Razor file.
:::

## Related Pages
- [Razor Panels Overview](index.md)
