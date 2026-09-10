---
title: Define All Interactive States
impact: HIGH
tags: state, pointer, pressed, disabled, focus
---

## Define All Interactive States

Custom interactive controls must define Normal, PointerOver, Pressed, Disabled, and Focused states. Missing states create dead spots where the control feels unresponsive.

**Incorrect (only Normal and PointerOver — Pressed and Disabled feel broken):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```

**Correct (complete state set):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
    <VisualState x:Name="Pressed">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorTertiaryBrush}" />
            <Setter Target="Root.RenderTransform">
                <Setter.Value>
                    <ScaleTransform ScaleX="0.98" ScaleY="0.98" />
                </Setter.Value>
            </Setter>
        </VisualState.Setters>
    </VisualState>
    <VisualState x:Name="Disabled">
        <VisualState.Setters>
            <Setter Target="Root.Opacity" Value="0.4" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```
