---
title: Exit Mirrors Entrance for Symmetry
impact: MEDIUM
tags: exit, entrance, symmetry, consistency
---

## Exit Mirrors Entrance for Symmetry

If an element enters by sliding up and fading in, it should exit by sliding down and fading out. Asymmetric enter/exit feels disorienting.

**Incorrect (enters from bottom, exits with scale — inconsistent):**

```xml
<!-- Entrance -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)"
                 From="40" To="0" Duration="0:0:0.25" />

<!-- Exit -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleX)"
                 From="1" To="0" Duration="0:0:0.25" />
```

**Correct (exit mirrors entrance — slide back down):**

```xml
<!-- Entrance -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)"
                 From="40" To="0" Duration="0:0:0.25" />

<!-- Exit -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)"
                 From="0" To="40" Duration="0:0:0.2" />
```
