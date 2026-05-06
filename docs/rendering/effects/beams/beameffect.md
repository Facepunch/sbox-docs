---
title: "BeamEffect"
icon: "〰️"
created: 2025-06-26
updated: 2025-06-28
---

# BeamEffect

With the BeamEffect you have basically got a LineRenderer that can be animated. You can have it make multiple beams, fade them out over time, scale over time and scroll and scale the texture over time. 

It also complies with [TemporaryEffect](/scene/components/reference/temporaryeffect.md), which means that it's perfect for creating things like tracers or lasers.


![BeamEffect animation demo](./images/beam-effect-demo.mp4)

## Follow Points

If enabled the beams will follow the points when they move. If you move the GameObject, the beam will move with it. If not they'll stay where they are for the duration of their lives.
