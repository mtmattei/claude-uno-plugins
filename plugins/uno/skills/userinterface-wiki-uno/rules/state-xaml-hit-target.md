---
title: Expand Hit Targets with Transparent Fill
impact: HIGH
tags: state, hit-target, touch, pointer, accessibility
---

## Expand Hit Targets with Transparent Fill

Small interactive elements need expanded hit areas. In CSS you'd use pseudo-element padding. In XAML, use a transparent Border or set Padding on the clickable container. Minimum 32x32px for touch (44x44 recommended).

**Incorrect (16px icon button — impossible to tap on mobile):**

```xml
<Button Padding="0" MinWidth="0" MinHeight="0">
    <FontIcon Glyph="&#xE72D;" FontSize="16" />
</Button>
```

**Correct (expanded touch target with padding):**

```xml
<Button Padding="12" MinWidth="44" MinHeight="44">
    <FontIcon Glyph="&#xE72D;" FontSize="16" />
</Button>
```
