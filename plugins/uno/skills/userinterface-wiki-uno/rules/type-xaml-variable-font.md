---
title: Use Variable Fonts for Continuous Weight
impact: MEDIUM
tags: type, variable-font, weight
---

## Use Variable Fonts for Continuous Weight

Variable fonts contain all weights in a single file, reducing bundle size and enabling smooth weight transitions. Use a single .ttf and set any FontWeight value from 100-900.

**Incorrect (loading 4 separate static font files):**

```xml
<FontFamily x:Key="AppFontLight">/Assets/Fonts/Inter-Light.ttf#Inter</FontFamily>
<FontFamily x:Key="AppFontRegular">/Assets/Fonts/Inter-Regular.ttf#Inter</FontFamily>
<FontFamily x:Key="AppFontMedium">/Assets/Fonts/Inter-Medium.ttf#Inter</FontFamily>
<FontFamily x:Key="AppFontBold">/Assets/Fonts/Inter-Bold.ttf#Inter</FontFamily>
```

**Correct (single variable font, any weight):**

```xml
<FontFamily x:Key="AppFont">/Assets/Fonts/Inter-Variable.ttf#Inter</FontFamily>

<TextBlock FontFamily="{StaticResource AppFont}" FontWeight="350" />
<TextBlock FontFamily="{StaticResource AppFont}" FontWeight="600" />
```
