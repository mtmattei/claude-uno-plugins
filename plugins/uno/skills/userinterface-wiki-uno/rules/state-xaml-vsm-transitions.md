---
title: Animate Between Visual States with Transitions
impact: MEDIUM
tags: state, vsm, transition, animation
---

## Animate Between Visual States with Transitions

VisualState.Setters snap instantly. For smooth state changes, add VisualTransitions with duration between states.

**Incorrect (background snaps on hover — jarring):**

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

**Correct (smooth transition between states):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualStateGroup.Transitions>
        <VisualTransition GeneratedDuration="0:0:0.15" />
    </VisualStateGroup.Transitions>
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```
