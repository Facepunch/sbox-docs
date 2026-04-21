---
title: "Screen-Space Reflections"
icon: "🪞"
created: 2025-07-28
updated: 2025-07-28
---

# Screen-Space Reflections

SSR is done in three steps

* Intersection
  * Uses the [G-Buffer] and [Screen Space Tracing] right after we do the Depth Pre-Pass, uses the result from last frame reprojected with [Motion]
  * ![](./images/screen-space-reflections-1.png)
* Denoising
  * Accumulates the result and makes it pretty taking off the rough edges
  * ![](./images/screen-space-reflections.png)
* Compositing
  * Sets the texture to be composited with [Dynamic Reflections]
  * ![](./images/screen-space-reflections-2.png)
