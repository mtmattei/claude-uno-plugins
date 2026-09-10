---
title: Semi-Transparent Borders Adapt to Any Background
impact: MEDIUM
tags: visual, border, opacity, brush, theme
---

## Semi-Transparent Borders Adapt to Any Background

Hard-coded border colors break on theme or background changes. Use semi-transparent brushes so borders adapt automatically.

**Incorrect (hard-coded gray — wrong on dark theme):**

```xml
<Border BorderBrush="#CCCCCC" BorderThickness="1">
    <TextBlock Text="Card" />
</Border>
```

**Correct (semi-transparent adapts to theme):**

```xml
<Border BorderBrush="{ThemeResource CardStrokeColorDefaultBrush}"
        BorderThickness="1">
    <TextBlock Text="Card" />
</Border>
```

Or with a custom alpha brush:

```xml
<SolidColorBrush x:Key="SubtleBorder" Color="{ThemeResource SystemBaseHighColor}" Opacity="0.1" />
```
