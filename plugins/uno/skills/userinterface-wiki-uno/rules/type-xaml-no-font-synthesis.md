---
title: Avoid Faux Bold and Italic
impact: HIGH
tags: type, font, synthesis, weight
---

## Avoid Faux Bold and Italic

Never rely on the platform to synthesize bold or italic from a regular font face. Faux bold thickens strokes uniformly (looks bloated) and faux italic just skews glyphs (looks wrong). Load the actual font weight/style file.

**Incorrect (requesting SemiBold when only Regular is loaded — platform synthesizes):**

```xml
<FontFamily x:Key="AppFont">/Assets/Fonts/Inter-Regular.ttf#Inter</FontFamily>

<TextBlock Text="Important" FontFamily="{StaticResource AppFont}"
           FontWeight="SemiBold" />
```

**Correct (load the actual SemiBold face):**

```xml
<FontFamily x:Key="AppFont">/Assets/Fonts/Inter-SemiBold.ttf#Inter</FontFamily>

<TextBlock Text="Important" FontFamily="{StaticResource AppFont}"
           FontWeight="SemiBold" />
```
