---
title: "SSAO"
icon: "🕳️"
created: 2024-12-05
updated: 2026-09-03
---

# SSAO

If your custom shader implements custom lighting you can sample the screen space ambient occlusion value for your lighting calculations.

```cpp
float ScreenSpaceAmbientOcclusion::Sample( float4 ScreenPosition )
```

![](./images/ssao.png) ![](./images/ssao-1.png)

Returns `1.0` when no ambient occlusion has been rendered this frame.
