---
title: Use Spring NaturalMotion for Organic Layout
impact: MEDIUM
tags: layout, spring, natural-motion, composition
---

## Use Spring NaturalMotion for Organic Layout

SpringVector3NaturalMotionAnimation creates organic, interruptible motion. Use it for layout animations that should feel physical and responsive.

**Incorrect (linear keyframe animation — feels mechanical):**

```csharp
var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(300);
```

**Correct (spring motion — organic and interruptible):**

```csharp
var spring = compositor.CreateSpringVector3Animation();
spring.FinalValue = new Vector3(0);
spring.DampingRatio = 0.7f;
spring.Period = TimeSpan.FromMilliseconds(80);

var group = compositor.CreateImplicitAnimationCollection();
group["Offset"] = spring;
visual.ImplicitAnimations = group;
```
