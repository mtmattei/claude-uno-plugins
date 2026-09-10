---
title: Use VisualStateManager for State-Driven Styling
impact: HIGH
tags: state, vsm, visual-state, styling
---

## Use VisualStateManager for State-Driven Styling

Use VisualStateManager instead of code-behind property changes for interactive states. VSM centralizes state definitions, enables animation, and works with adaptive triggers.

**Incorrect (toggling properties in code-behind — scattered, not animatable):**

```csharp
private void OnPointerEntered(object sender, PointerRoutedEventArgs e)
{
    MyBorder.Background = new SolidColorBrush(Colors.LightBlue);
    MyBorder.BorderThickness = new Thickness(2);
}
```

**Correct (VisualStateManager in XAML — declarative and animatable):**

```xml
<Border x:Name="MyBorder" PointerEntered="OnPointerEntered">
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup x:Name="HoverStates">
            <VisualState x:Name="Normal" />
            <VisualState x:Name="PointerOver">
                <VisualState.Setters>
                    <Setter Target="MyBorder.Background"
                            Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
                    <Setter Target="MyBorder.BorderThickness" Value="2" />
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>
</Border>
```
