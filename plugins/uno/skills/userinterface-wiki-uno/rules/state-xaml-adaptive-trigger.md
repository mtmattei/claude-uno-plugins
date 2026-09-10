---
title: Use AdaptiveTrigger for Responsive States
impact: HIGH
tags: state, adaptive-trigger, responsive, breakpoint
---

## Use AdaptiveTrigger for Responsive States

AdaptiveTrigger replaces CSS media queries in XAML. Use it to switch visual states based on window width, replacing code-behind SizeChanged handlers.

**Incorrect (manual breakpoint logic in code-behind):**

```csharp
private void OnSizeChanged(object sender, SizeChangedEventArgs e)
{
    if (e.NewSize.Width < 640)
    {
        SidePanel.Visibility = Visibility.Collapsed;
        BottomNav.Visibility = Visibility.Visible;
    }
    else
    {
        SidePanel.Visibility = Visibility.Visible;
        BottomNav.Visibility = Visibility.Collapsed;
    }
}
```

**Correct (declarative adaptive triggers):**

```xml
<VisualStateGroup x:Name="LayoutStates">
    <VisualState x:Name="Wide">
        <VisualState.StateTriggers>
            <AdaptiveTrigger MinWindowWidth="640" />
        </VisualState.StateTriggers>
        <VisualState.Setters>
            <Setter Target="SidePanel.Visibility" Value="Visible" />
            <Setter Target="BottomNav.Visibility" Value="Collapsed" />
        </VisualState.Setters>
    </VisualState>
    <VisualState x:Name="Narrow">
        <VisualState.StateTriggers>
            <AdaptiveTrigger MinWindowWidth="0" />
        </VisualState.StateTriggers>
        <VisualState.Setters>
            <Setter Target="SidePanel.Visibility" Value="Collapsed" />
            <Setter Target="BottomNav.Visibility" Value="Visible" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```
