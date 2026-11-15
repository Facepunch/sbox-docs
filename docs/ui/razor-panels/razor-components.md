---
title: "Razor Components"
icon: "📝"
created: 2024-09-24
updated: 2026-11-15
---

# Razor Components

Razor Component allow you to define some content to show inside another razor file. This makes it easy to create Panels that are more agile and reusable.

# Creating a Razor Component

Here's an example called `InfoCard.razor`:

```csharp
<root>
    <div class="header">@Header</div>
    <div class="body">@Body</div>
    <div class="footer">@Footer</div>
</root>

@code
{
    public RenderFragment Header { get; set; }
    public RenderFragment Body { get; set; }
    public RenderFragment Footer { get; set; }
}
```

# Using a Razor Component

We can now very quickly and easily create Info Cards, and even add events/functions that interact with the parent Panel:

```csharp
<root>
	<InfoCard>
        <Header>My Card</Header>
        <Body>
            <div class="title">This is my card</div>
        </Body>
        <Footer>
            <div class="button" onclick="@CloseCard">Close Card</div>
        </Footer>
    </InfoCard>
</root>

@code 
{
    void CloseCard()
    {
        Delete();
    }
}
```

# Elements inside Panels

All Panels have a render fragment called ChildContent, so you can add elements inside of any Panel without adding your own fragments:

```csharp
<InfoCard>
  <ChildContent>
    <label>Something inside of InfoCard</label>
  </ChildContent>
</InfoCard>
```

# Razor Components With Arguments

`RenderFragment<T>` is used to define a fragment that takes an argument. Here's `PlayerList.razor`:

```csharp
@using Sandbox;

<root>
    @foreach ( var player in Players )
    {
        <div class="row">
            @PlayerRow( player )
        </div>
    }
</root>

@code
{
    /// <summary>
    /// Players to display in the list.
    /// </summary>
    public List<Player> Players { get; set; }

    /// <summary>
    /// Content to render for each player.
    /// </summary>
    public RenderFragment<Player> PlayerRow { get; set; }
}
```

As you can see, we're now defining `PlayerRow` which takes an argument. To call it we just call it like a function, with the Player argument.

Now to use it we do this:

```csharp
@using Sandbox;

<root>
    <PlayerList Players=@MyPlayerList>
        <PlayerRow Context="item">
            @if (item is not Player player)
                return;

            <div class="row">@player.Name</div>
            <div class="row">@player.Health</div>
        </PlayerRow>
    </PlayerList>
</root>

@code 
{
    List<Player> MyPlayerList;
}
```

In the PlayerRow, we add the special attribute `Context`, which tells our code generator that this is going to need a special `object` context.

Then in the fragment, we can convert this `object` into our target type. Then you're free to use it as you wish.

# Generics

You can make a generic Panel class by using `@typeparam` to define the name of the parameter

```csharp
@typeparam T

<root>
    This is my class for @Value
</root>

@code
{
    public T Value { get; set; }
}
```

You will then be able to use this like a regular generics class in C#, like `PanelName<string>` etc.

In Razor, you will need to add a `T` attribute to define the type like so:

```csharp
<root>
	<MyPanel T="string"></MyPanel>
</root>
```
