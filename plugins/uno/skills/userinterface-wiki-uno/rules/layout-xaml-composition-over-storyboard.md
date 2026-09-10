---
title: Composition Animations Over Storyboard for Layout
impact: MEDIUM
tags: layout, composition, storyboard, performance
---

## Composition Animations Over Storyboard for Layout

Composition animations run on the compositor thread, independent of the UI thread. For layout-triggered animations (offset, scale, opacity), prefer composition over XAML Storyboards which run on the UI thread and can cause jank.

**Incorrect (XAML Storyboard for offset — runs on UI thread):**

```xml
<Storyboard x:Name="SlideIn">
    <DoubleAnimation Storyboard.TargetName="MyPanel"
                     Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.X)"
                     From="200" To="0" Duration="0:0:0.3" />
</Storyboard>
```

**Correct (composition animation — runs off UI thread):**

```csharp
var visual = ElementCompositionPreview.GetElementVisual(MyPanel);
var animation = visual.Compositor.CreateScalarKeyFrameAnimation();
animation.InsertKeyFrame(0f, 200f);
animation.InsertKeyFrame(1f, 0f, visual.Compositor.CreateCubicBezierEasingFunction(
    new Vector2(0, 0), new Vector2(0.2f, 1f)));
animation.Duration = TimeSpan.FromMilliseconds(300);
visual.StartAnimation("Translation.X", animation);
```
