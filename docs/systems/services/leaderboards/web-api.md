---
title: "Web Api"
icon: "📄"
created: 2024-08-23
updated: 2024-08-23
---

# Web Api

The API for leaderboards is publically accessible. Have fun!

```none
https://services.facepunch.com/sbox/package/leaderboard/{package_ident}/
```

### stat

The name of the stat to base the leaderboard on. This does not need to have been registered formally.

### count `(default 10)`

The amount of results to return

### offset `(default 0)`

If higher than 0 then the results will start at this number. If lower than 0, we'll start at the bottom of the results. For example, a value us -1 will return the very last page of results.

### aggregation `(default sum)`

Describes how to aggregate the stats. This is fundamental to getting the leaderboard in a format that makes sense.

| `sum` | Adds all values together. This is useful for things like the most kills, where you have a stat that is incremented on each kill. |
|-----|----------------------------------------------------------------------------------------------------------------------------------|
| `max` | The maximum single value every submitted. An example would be for high scores.                                                   |
| `min` | The lowest value every submitted. You could use this for fastest time etc.                                                       |
| `avg` | The average value from every value submitted. I can't think of a use case.                                                       |
| `last` | The last value submitted. I can't think of a use case.                                                                           |

### sort `(default desc)`

How the leaderboard values should be sorted.

| `desc` | Highest values first |
|------|----------------------|
| `asc` | Lowest values first  |

### datefilter `(default none)`

Allows querying a specific time period for a leaderboard. This allows creating things like weekly leaderboards.

| `none` | Do not limit by time period. |
|------|------------------------------|
| `year` | Limit by year                |
| `month` | Limit by month               |
| `week` | Limit by week                |
| `day` | Limit by day                 |

### date`(default now)`

Allows specifying a date to get for datefilter. You don't need to be exact with the date, it just has to be within the period you want to get. If unset, will default to now - which will get the current leaderboard for any datefilter.

This does nothing if datefilter is none.

### centersteamid`(default 0)`

Center the results on a certain SteamId within the board. Offset will offset from this position.

### country`(default none)`

Limit results to a specific country code. If set to `auto` this will return the current user's country, if logged in.

### friends `(default false)`

Limit results to the current logged user's friends.

### include\[\] `(default null)`

Allows a list of steamids to always show on the leaderboard, even if they're not in current subset. This is useful for situations where you want to show the top 20, but want the current player's score to be present too.

# Example

```none
https://services.facepunch.com/sbox/package/leaderboard/facepunch.ss1/?stat=zombies_killed
```

```json
{
  Stat: "zombies_killed",
  TotalEntries: 2588,
  Entries: 
  [
    {
      Rank: 1,
      Value: 645806,
      SteamId: 76561198043785940,
      CountryCode: "us",
      DisplayName: "puxorb",
      Timestamp: "2024-08-23T03:44:17.0174014+00:00",
      Data: { }
    },
    {
      Rank: 2,
      Value: 310250,
      SteamId: 76561198053736750,
      CountryCode: "au",
      DisplayName: "Rin",
      Timestamp: "2024-08-19T18:28:10.6930596+00:00",
      Data: { }
    }
  ],
  DateDescription: null,
}
```
