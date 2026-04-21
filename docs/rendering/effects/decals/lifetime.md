---
title: "Lifetime"
icon: "👶"
created: 2025-06-26
updated: 2025-06-26
---

# Lifetime

![](./images/lifetime.png)


The "life" properties on the decal component allow you to define how long it should live for. Any lifetime curve properties on the Decal will be played back using this time. You can randomise it to add variety.



:::info
With LifeTime - the Component will stop rendering at the end of its life, but it won't be deleted and it won't delete the GameObject. You should add a TemporaryEffect component to do this.

:::


## Looping

If the effect is looped, it'll restart once it gets to the end, forever. This would generally be used if you want to animate a render over its lifetime but have it loop rather than disappear.


## Transient 

If you mark your decal as Transient, then the oldest one will be deleted when maxdecals is reached. This is good for bullet holes, where you probably want them sticking around forever unless it's not practical.


## Static

If Lifetime is 0, or Looped is true, the Decal will be static. It will last forever. This is appropriate for world decals.
