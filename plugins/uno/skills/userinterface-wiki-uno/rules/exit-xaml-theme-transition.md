---
title: Use ThemeTransition for Standard Enter/Exit
impact: HIGH
tags: exit, theme-transition, entrance, collection
---

## Use ThemeTransition for Standard Enter/Exit

WinUI provides built-in ThemeTransitions that handle entrance and exit automatically. Use them instead of writing custom Storyboards for standard patterns.

**Incorrect (no transitions — elements pop in/out abruptly):**

```xml
<StackPanel>
    <TextBlock Text="Welcome" />
</StackPanel>
```

**Correct (theme transitions for smooth entrance):**

```xml
<StackPanel>
    <StackPanel.ChildrenTransitions>
        <EntranceThemeTransition />
    </StackPanel.ChildrenTransitions>
    <TextBlock Text="Welcome" />
</StackPanel>
```
