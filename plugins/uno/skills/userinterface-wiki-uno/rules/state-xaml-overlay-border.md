---
title: Use Overlay Shapes Instead of Extra Elements
impact: MEDIUM
tags: state, overlay, pseudo-element, decoration
---

## Use Overlay Shapes Instead of Extra Elements

In CSS, ::before/::after create decorative overlays without DOM nodes. In XAML, use a Border or Rectangle in the same Grid cell to achieve the same effect without adding a wrapper element.

**Incorrect (extra wrapping StackPanel just for a highlight overlay):**

```xml
<StackPanel>
    <Border Background="Gold" Opacity="0.3" Height="40" />
    <TextBlock Text="Featured" Margin="0,-40,0,0" />
</StackPanel>
```

**Correct (overlay in same Grid cell — clean and layered):**

```xml
<Grid>
    <Border Background="{ThemeResource SystemAccentColorLight2}"
            Opacity="0.15"
            CornerRadius="4" />
    <TextBlock Text="Featured"
               Padding="8,4"
               VerticalAlignment="Center" />
</Grid>
```
