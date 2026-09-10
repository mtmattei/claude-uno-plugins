---
title: Concentric CornerRadius for Nested Elements
impact: HIGH
tags: visual, corner-radius, nesting, concentric
---

## Concentric CornerRadius for Nested Elements

Inner CornerRadius = outer CornerRadius minus padding. This creates concentric curves that look intentional. Matching inner/outer radius looks wrong because the curves don't align.

**Incorrect (same radius on both — inner curve looks too round):**

```xml
<Border CornerRadius="16" Padding="8"
        Background="{ThemeResource CardBackgroundFillColorDefaultBrush}">
    <Border CornerRadius="16"
            Background="{ThemeResource SolidBackgroundFillColorBaseBrush}">
        <TextBlock Text="Content" Padding="12" />
    </Border>
</Border>
```

**Correct (inner radius = 16 - 8 = 8):**

```xml
<Border CornerRadius="16" Padding="8"
        Background="{ThemeResource CardBackgroundFillColorDefaultBrush}">
    <Border CornerRadius="8"
            Background="{ThemeResource SolidBackgroundFillColorBaseBrush}">
        <TextBlock Text="Content" Padding="12" />
    </Border>
</Border>
```
