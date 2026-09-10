---
title: Add Slight Delay for Catching-Up Feel
impact: LOW
tags: layout, delay, stagger, natural
---

## Add Slight Delay for Catching-Up Feel

A small delay (30-60ms) before a layout animation starts creates a natural "catching up" feel, as if the element reacts to the change rather than predicting it.

**Incorrect (animation starts simultaneously with trigger — feels robotic):**

```csharp
var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(250);
animation.DelayTime = TimeSpan.Zero;
```

**Correct (small delay for organic feel):**

```csharp
var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(250);
animation.DelayTime = TimeSpan.FromMilliseconds(40);
animation.DelayBehavior = AnimationDelayBehavior.SetInitialValueBeforeDelay;
```
