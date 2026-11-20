---
title: "Senses & Relationships"
icon: "👀"
created: 2026-11-20
updated: 2026-11-20
---

# Senses & Relationships

Add a `SensesLayer` to let an NPC notice its surroundings. **Targets** selects players, NPCs, or both. **Sight Range** and **Field Of View** control sight. `BaseCombatNpc` also uses **Hearing Range** and **Quiet Speed** to notice moving hostiles.

Sensing updates periodically on the host. Read the results through `Npc.Senses`; you don't need to run a scan yourself. Disabling the layer clears its results.

# Factions

A faction is a string identifying an NPC to others. The built-in `Factions` include `Player`, `Ally`, `Enemy`, `Citizen` and `Monster`. You can use your own names too.

A relationship describes how one NPC regards another: `Friendly`, `Neutral`, `Hostile` or `Fearful`. Relationships are directional. A guard can be friendly to a thief who is hostile to it.

Set `Faction` in the inspector, or override it in code. Override `SetupRelationships()` to declare who this NPC likes and dislikes:

```csharp
protected override void SetupRelationships()
{
    SetRelationship( Factions.Player, Disposition.Friendly );
    SetRelationship( "bandits", Disposition.Hostile );
    SetRelationship( Factions.Monster, Disposition.Fearful );
}
```

Call `base.SetupRelationships()` first if you want to keep the built-in relationships for the NPC's faction. Without an override, the base uses those defaults.

For a particular GameObject, use `npc.SetRelationship( other, Disposition.Hostile )`. This takes precedence over its faction. `ClearRelationship( other )` removes that override.

If relationships depend on game state, override `GetFallbackRelationship( GameObject other )`. It runs only when no explicit entity or faction relationship exists, and returns neutral by default. Keep it cheap: sensing calls it for potential targets.

# Finding targets

```csharp
var enemy = Senses.FindBestTarget();
var threat = Senses.FindNearestVisible( Disposition.Fearful );
```

`FindBestTarget()` picks a visible hostile by relationship priority, then distance. `VisibleHostiles` and `VisibleThreats` expose the current lists. `FindNearestVisible()` also accepts a tag; it can run visibility traces.

Targets need an active `Component.IDamageable`.

![An NPC stops attacking when its target's damageable component is disabled. 1920x1080](images/live-targets.mp4)

# Disturbances

Emit a stimulus on the host to tell nearby NPCs something happened. It doesn't need to play an actual sound.

```csharp
Scene.GetSystem<NpcSystem>().EmitStimulus(
    StimulusKinds.Gunshot, WorldPosition, GameObject,
    radius: 1024f, lifetime: 1f );
```

Inside an NPC, `EmitStimulus( StimulusKinds.Gunshot )` is the shorter version. Kinds are strings, so your game can add its own.

`Senses.Disturbance` contains the nearest active stimulus within that stimulus's radius, ignoring the NPC's own. `HearingRange` doesn't apply. Your schedule decides how to react.

![Idle NPCs investigate a gunshot stimulus. 1920x1080](images/stimulus.mp4)
