---
title: Consistent Shadow Direction
impact: MEDIUM
tags: visual, shadow, direction, light-source
---

## Consistent Shadow Direction

All shadows in the app should share the same offset direction (single virtual light source). ThemeShadow handles this automatically — avoid mixing it with custom DropShadow that has a different offset.

**Incorrect (custom shadow with different light angle than ThemeShadow):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Offset = new Vector3(-4, -4, 0); // Light from bottom-right
// Meanwhile ThemeShadow assumes light from top
```

**Correct (stick to ThemeShadow or use consistent custom offsets):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Offset = new Vector3(0, 2, 0); // Light from top, matching system
shadow.BlurRadius = 8;
shadow.Color = Color.FromArgb(40, 0, 0, 0);
```
