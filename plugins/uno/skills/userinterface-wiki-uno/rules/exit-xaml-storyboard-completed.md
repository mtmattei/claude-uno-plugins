---
title: Handle Storyboard.Completed Before Removing Elements
impact: HIGH
tags: exit, storyboard, completed, removal
---

## Handle Storyboard.Completed Before Removing Elements

When using a custom exit animation via Storyboard, never remove the element from the visual tree until the Storyboard fires Completed. Removing early aborts the animation.

**Incorrect (removes element immediately — animation never plays):**

```csharp
private void RemoveItem(UIElement element)
{
    var fadeOut = new DoubleAnimation
    {
        To = 0, Duration = TimeSpan.FromMilliseconds(200)
    };
    Storyboard.SetTarget(fadeOut, element);
    Storyboard.SetTargetProperty(fadeOut, "Opacity");

    var sb = new Storyboard { Children = { fadeOut } };
    sb.Begin();
    Panel.Children.Remove(element); // Too early!
}
```

**Correct (waits for Completed then removes):**

```csharp
private void RemoveItem(UIElement element)
{
    var fadeOut = new DoubleAnimation
    {
        To = 0, Duration = TimeSpan.FromMilliseconds(200)
    };
    Storyboard.SetTarget(fadeOut, element);
    Storyboard.SetTargetProperty(fadeOut, "Opacity");

    var sb = new Storyboard { Children = { fadeOut } };
    sb.Completed += (_, _) => Panel.Children.Remove(element);
    sb.Begin();
}
```
