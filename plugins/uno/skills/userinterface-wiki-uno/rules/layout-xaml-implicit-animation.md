---
title: Use Implicit Animations for Layout Changes
impact: HIGH
tags: layout, implicit-animation, composition, offset
---

## Use Implicit Animations for Layout Changes

Composition implicit animations automatically animate property changes on a Visual. Use ElementCompositionPreview to attach them so elements glide to new positions instead of snapping.

**Incorrect (elements jump to new positions after layout change):**

```csharp
// No animation — items snap when reordered
myElement.Margin = new Thickness(100, 0, 0, 0);
```

**Correct (implicit animation on Offset — smooth repositioning):**

```csharp
var visual = ElementCompositionPreview.GetElementVisual(myElement);
var compositor = visual.Compositor;

var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(250);

var group = compositor.CreateImplicitAnimationCollection();
group["Offset"] = animation;
visual.ImplicitAnimations = group;
```
