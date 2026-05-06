---
title: "Achievements"
icon: "🏅"
created: 2024-09-10
updated: 2024-09-10
---

# Achievements

Your game can have multiple achievements for players to unlock.

## Score

Each achievement can have a different score. The score is usually between 5 and 100, which is really something you need to choose based on the achievement. Generally, give low values to easy-to-achieve things and a high value to hard-to-achieve things. Have many low value, and few high value.

When a player unlocks the achievement, the score is added to their global score.

You have full control over choosing the score but the total combined for your game cannot exceed 1000.


## Icons

The icon is automatically resized to 128x128. It will generally be rendered quite small, next to the name of the achievement, so should be treated more like an icon than a picture.


## Stat Based

When your achievement unlock mode is set to "Stat" it will automatically be unlocked for you. All you need to do is make sure the stat is set up properly.

An example of a stat based achievement would be the "100 coins".

You would do `Stat.Increment( "coins", 1 )` in your code every time you collected a coin, then you can set your achievement to this..

| Property | Value | Explanation |
|----------|-------|-------------|
| Target Stat | "coin" |             |
| Aggregation | Sum   |             |
| Min Value | 0     |             |
| Max Value | 100   |             |
| Show Progress | true  |             |

* Target Stat: "coin" (the name of the stat)
* Aggregation: Sum (you want to add the 1's values together)
* Min Value: 0, Max Value: 100 (unlocks at 100)
* Show Progress - yes - will show progress between 0 and 100 as a percentage


## Manual

If the achievement is set to manual, then you can unlock it in your code like this:

```csharp
Sandbox.Services.Achievements.Unlock( "achievement_ident" );
```


# Listing


You can get the list of achievements at any time in your game

```csharp
foreach ( var a in Sandbox.Services.Achievements.All )
{
    // do something
}
```


Or from outside the game, you can access it via the API

```markup
https://services.facepunch.com/sbox/achievement/list?package=facepunch.testbed
```
