---
title: Use Uno Toolkit ShadowContainer for Layered Shadows
impact: MEDIUM
tags: visual, shadow, uno-toolkit, layered
---

## Use Uno Toolkit ShadowContainer for Layered Shadows

Multiple layered shadows create more realistic depth than a single shadow. Uno Toolkit's ShadowContainer supports multiple shadow definitions in XAML.

**Incorrect (single flat shadow — lacks depth):**

```xml
<Border Translation="0,0,16">
    <Border.Shadow>
        <ThemeShadow />
    </Border.Shadow>
    <TextBlock Text="Card" />
</Border>
```

**Correct (layered shadows for nuanced depth):**

```xml
<utu:ShadowContainer>
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow BlurRadius="2" OffsetY="1"
                        Color="#20000000" />
            <utu:Shadow BlurRadius="8" OffsetY="4"
                        Color="#14000000" />
            <utu:Shadow BlurRadius="24" OffsetY="8"
                        Color="#0A000000" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Border CornerRadius="8" Padding="16"
            Background="{ThemeResource CardBackgroundFillColorDefaultBrush}">
        <TextBlock Text="Card" />
    </Border>
</utu:ShadowContainer>
```
