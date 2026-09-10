---
title: Lightweight Styling Over Full Template Override
impact: HIGH
tags: state, lightweight-styling, resource, theme
---

## Lightweight Styling Over Full Template Override

Override specific theme resources to restyle controls instead of copying entire ControlTemplates. Lightweight styling survives framework updates and reduces XAML bloat.

**Incorrect (copying entire Button template just to change background):**

```xml
<Style x:Key="AccentButton" TargetType="Button">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <!-- 80+ lines of copied template just to change one brush -->
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

**Correct (override just the resource key):**

```xml
<Button Content="Save">
    <Button.Resources>
        <SolidColorBrush x:Key="ButtonBackground"
                         Color="{ThemeResource SystemAccentColor}" />
        <SolidColorBrush x:Key="ButtonBackgroundPointerOver"
                         Color="{ThemeResource SystemAccentColorLight1}" />
        <SolidColorBrush x:Key="ButtonBackgroundPressed"
                         Color="{ThemeResource SystemAccentColorDark1}" />
    </Button.Resources>
</Button>
```
