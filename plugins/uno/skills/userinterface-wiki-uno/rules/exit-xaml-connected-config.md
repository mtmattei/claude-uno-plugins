---
title: Configure ConnectedAnimation Easing
impact: MEDIUM
tags: exit, connected-animation, spring, easing
---

## Configure ConnectedAnimation Easing

ConnectedAnimation defaults to a system curve, but for gesture-driven transitions use a spring configuration. Match the animation personality to the interaction type.

**Incorrect (default easing for a drag-to-dismiss — feels stiff):**

```csharp
var animation = ConnectedAnimationService.GetForCurrentView()
    .GetAnimation("card");
animation?.TryStart(TargetElement);
```

**Correct (gravity spring for natural gesture follow-through):**

```csharp
var animation = ConnectedAnimationService.GetForCurrentView()
    .GetAnimation("card");
if (animation is not null)
{
    animation.Configuration = new DirectConnectedAnimationConfiguration();
    animation.TryStart(TargetElement);
}
```
