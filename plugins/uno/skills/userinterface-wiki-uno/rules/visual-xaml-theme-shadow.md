---
title: Use ThemeShadow for Elevation
impact: HIGH
tags: visual, shadow, elevation, depth
---

## Use ThemeShadow for Elevation

ThemeShadow provides system-consistent elevation that adapts to light/dark theme. Use it with Translation.Z to set elevation level rather than hardcoding shadow parameters.

**Incorrect (no elevation — card floats without visual grounding):**

```xml
<Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
        CornerRadius="8" Padding="16">
    <TextBlock Text="Card content" />
</Border>
```

**Correct (ThemeShadow with Z translation):**

```xml
<Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
        CornerRadius="8" Padding="16"
        Translation="0,0,32">
    <Border.Shadow>
        <ThemeShadow />
    </Border.Shadow>
    <TextBlock Text="Card content" />
</Border>
```
