---
title: Disable Color Fonts When Inappropriate
impact: LOW
tags: type, color-font, emoji
---

## Disable Color Fonts When Inappropriate

IsColorFontEnabled defaults to true, which renders color emoji and color glyphs. In monochrome icon contexts or data-dense UIs, disable it to keep a consistent visual weight.

**Incorrect (color emoji in a monochrome toolbar label):**

```xml
<TextBlock Text="&#x2764; Favorites"
           Foreground="{ThemeResource OnSurfaceBrush}" />
```

**Correct (monochrome glyph matches surrounding icons):**

```xml
<TextBlock Text="&#x2764; Favorites"
           Foreground="{ThemeResource OnSurfaceBrush}"
           IsColorFontEnabled="False" />
```
