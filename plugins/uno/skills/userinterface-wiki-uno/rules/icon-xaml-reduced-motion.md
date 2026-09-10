---
title: Respect Reduced Motion for Icon Animations
impact: HIGH
tags: icon, accessibility, reduced-motion
---

## Respect Reduced Motion for Icon Animations

Check UISettings.AnimationsEnabled before playing icon animations. When disabled, show the final state immediately via the fallback icon.

**Incorrect (always plays animation — ignores user preference):**

```csharp
private async Task AnimateIcon()
{
    await IconPlayer.PlayAsync(0, 1, looped: false);
}
```

**Correct (skip animation when reduced motion is set):**

```csharp
private async Task AnimateIcon()
{
    var settings = new UISettings();
    if (settings.AnimationsEnabled)
    {
        await IconPlayer.PlayAsync(0, 1, looped: false);
    }
    else
    {
        // Jump to final frame
        IconPlayer.SetProgress(1.0);
    }
}
```
