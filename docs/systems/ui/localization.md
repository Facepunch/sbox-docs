---
title: "Localization"
icon: "💬"
created: 2024-09-24
updated: 2024-09-24
sources:
  - engine/Sandbox.Engine/Systems/Localization/Language.cs
---

# Localization

> **New to UI?** Reference for the localization system; UI overview is at **[UI](index.md)**.

Localization allows you to easily support multiple languages by replacing hardcoded text with translation tokens that automatically resolve to the user's selected language.

## Quick Working Example

Create a JSON file for your translations (e.g., `Localization/en/sandbox.json`):

```json
{
  "menu.helloworld": "Hello World",
  "player.health": "Health: {amount}"
}
```

Use the tokens in your Razor UI by prefixing them with `#`, or grab them in C# directly:

```csharp
@using Sandbox;
@inherits PanelComponent

<root>
    <!-- Automatic UI translation -->
    <label>#menu.helloworld</label>
</root>

@code
{
    protected override void OnStart()
    {
        // Manual C# translation with data insertion
        var data = new Dictionary<string, object> { { "amount", 100 } };
        Log.Info( Language.GetPhrase( "player.health", data ) );
    }
}
```

### Razor UI Tokens

Any text inside a UI component (like a `<label>`) that starts with a `#` is treated as a localization token.

```markup
<!-- Looks for the "spawnmenu.props" key in the active JSON file -->
<label>#spawnmenu.props</label>
```

### Accessing Phrases in C#

You can retrieve translated strings in code using `Language.GetPhrase()`. If the token isn't found, it simply returns the token string itself.

```csharp
string greeting = Language.GetPhrase( "menu.helloworld" );
```

### Passing Data to Phrases

Translations often require dynamic data, like a player's name or a score. You can embed variables in your JSON strings using curly braces `{VariableName}`.

```json
{
  "killfeed.killed": "{Attacker} killed {Victim}!"
}
```

You can pass these variables through C# using a `Dictionary`:

```csharp
var data = new Dictionary<string, object>()
{
    { "Attacker", "Alice" },
    { "Victim", "Bob" }
};

string message = Language.GetPhrase( "killfeed.killed", data );
// Returns: "Alice killed Bob!"
```

## Supported Languages

The user can choose their language in the game's options menu. Your translation files should be placed in folders matching the codes below (e.g., `Localization/fr/ui.json`).

| English Name | Language Code | English Name | Language Code |
|--------------|---------------|--------------|---------------|
| Arabic       | `ar`          | Norwegian    | `no`          |
| Bulgarian    | `bg`          | Pirate       | `en-pt`       |
| Simp. Chinese| `zh-cn`       | Polish       | `pl`          |
| Trad. Chinese| `zh-tw`       | Portuguese   | `pt`          |
| Czech        | `cs`          | Pt-Brazil    | `pt-br`       |
| Danish       | `da`          | Romanian     | `ro`          |
| Dutch        | `nl`          | Russian      | `ru`          |
| English      | `en`          | Spanish      | `es`          |
| Finnish      | `fi`          | Sp-LatAm     | `es-419`      |
| French       | `fr`          | Swedish      | `sv`          |
| German       | `de`          | Thai         | `th`          |
| Greek        | `el`          | Turkish      | `tr`          |
| Hungarian    | `hu`          | Ukrainian    | `uk`          |
| Italian      | `it`          | Vietnamese   | `vn`          |
| Japanese     | `ja`          | Korean       | `ko`          |

## Troubleshooting

:::danger "My text shows up as #menu.title instead of the translation!"
This means the engine couldn't find the token in any of the loaded JSON files for the current language. Ensure your JSON file is formatted correctly, is located in the right language folder, and the token is spelled exactly the same.
:::

:::warning "Variables in my text aren't being replaced!"
Ensure the dictionary keys you pass into `Language.GetPhrase()` exactly match the variable names inside the curly braces in your JSON file (they are case-sensitive).
:::

:::tip Debugging Localization
You can set the console variable `lang.showkeys` to `true` (or toggle `Language.DisplayKeys` in code). This will force the UI to display the raw `#token` instead of the translation, making it easy to identify which strings are localized and which are hardcoded.
:::

## Related Pages
* [UI Overview](index.md)
* [Razor Panels](razor-panels/index.md)
