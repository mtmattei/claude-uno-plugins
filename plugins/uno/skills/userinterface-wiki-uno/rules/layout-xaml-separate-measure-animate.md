---
title: Separate Measurement from Animation Target
impact: HIGH
tags: layout, measure, animate, feedback-loop
---

## Separate Measurement from Animation Target

Never measure and animate the same element — it creates a feedback loop. Use one element for natural size measurement and a parent for the animated bounds.

**Incorrect (animating Height on the element being measured):**

```csharp
private void OnContentSizeChanged(object sender, SizeChangedEventArgs e)
{
    // Animating Height triggers another SizeChanged — infinite loop
    AnimateHeight(ContentPanel, e.NewSize.Height);
}
```

**Correct (inner element measures, outer element animates):**

```xml
<Border x:Name="AnimatedWrapper">
    <StackPanel x:Name="MeasuredContent"
                SizeChanged="OnContentSizeChanged">
        <!-- Dynamic content here -->
    </StackPanel>
</Border>
```

```csharp
private void OnContentSizeChanged(object sender, SizeChangedEventArgs e)
{
    // Animate the wrapper, not the measured content
    AnimateHeight(AnimatedWrapper, e.NewSize.Height);
}
```
