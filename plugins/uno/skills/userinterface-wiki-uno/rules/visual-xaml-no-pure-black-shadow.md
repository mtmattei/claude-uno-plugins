---
title: No Pure Black Shadows
impact: MEDIUM
tags: visual, shadow, color, opacity
---

## No Pure Black Shadows

Pure black shadows (#000) look harsh and unnatural. Use semi-transparent neutral colors that adapt to the background.

**Incorrect (pure black shadow — too harsh):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Color = Colors.Black;
shadow.BlurRadius = 12;
```

**Correct (semi-transparent neutral — soft and natural):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Color = Color.FromArgb(40, 0, 0, 0); // 15% opacity
shadow.BlurRadius = 12;
```
