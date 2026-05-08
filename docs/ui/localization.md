---
title: "Localization"
icon: "💬"
created: 2024-09-24
updated: 2024-09-24
---

# Localization

When displaying text such as `Hello World`, you should instead use a localization token like `#menu.helloworld`. This will automatically replace the text with the corresponding language set by the user when set on a Label, allowing you to easily support multiple languages.

# Tokens

In UI, any displayed string that begins with a `#` will be recognized as a token, which means it will look for it's real value in the localization system.

# Example

Filename: `Localization/en/sandbox.json`

```json
{
  "menu.helloworld": "Hello World",
  "spawnmenu.props": "Models",
  "spawnmenu.tools": "Tools"
  "spawnmenu.cloud": "sbox.game",
}
```

Then it's as easy as doing the following:

```markup
<label>#spawnmenu.props</label>
```

# Advanced tokens

Sometimes you need to display text which has to be mixed with names of players, other items and so on. You will need to use a more advanced technique to display those strings.

First, in your localization file, add placeholders using braces like this:

```json
  "player.join": "{Name} has joined the game",
  "player.leave": "{Name} has left the game",
```

Then, use Language.GetPhrase to substitute that placeholder with your desired text:

```cs
    var data = new Dictionary<string, object>();
    data["Name"] = connection.DisplayName;
    var message = Language.GetPhrase( "player.join", data );
```

Make sure to tell your localization team to leave the text inside the placeholder intact, even in different languages.

# Languages

| English Name | API Language Code |
|--------------|-------------------|
| Arabic       | ar                |
| Bulgarian    | bg                |
| Simplified Chinese | zh-cn             |
| Traditional Chinese | zh-tw             |
| Czech        | cs                |
| Danish       | da                |
| Dutch        | nl                |
| English      | en                |
| Finnish      | fi                |
| French       | fr                |
| German       | de                |
| Greek        | el                |
| Hungarian    | hu                |
| Italian      | it                |
| Japanese     | ja                |
| Korean       | ko                |
| Norwegian    | no                |
| Pirate       | en-pt             |
| Polish       | pl                |
| Portuguese   | pt                |
| Portuguese-Brazil | pt-br             |
| Romanian     | ro                |
| Russian      | ru                |
| Spanish-Spain | es                |
| Spanish-Latin America | es-419            |
| Swedish      | sv                |
| Thai         | th                |
| Turkish      | tr                |
| Ukrainian    | uk                |
| Vietnamese   | vn                |
