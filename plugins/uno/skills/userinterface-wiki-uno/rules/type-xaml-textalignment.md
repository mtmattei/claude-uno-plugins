---
title: Text Alignment Matches Reading Direction
impact: MEDIUM
tags: type, alignment, rtl, localization
---

## Text Alignment Matches Reading Direction

Use TextAlignment="DetectFromContent" or leave at default (Start) for body text. Never hard-code Left for localizable content — it breaks RTL layouts.

**Incorrect (hard-coded Left breaks Arabic/Hebrew):**

```xml
<TextBlock Text="{x:Bind Description}"
           TextAlignment="Left" />
```

**Correct (Start follows FlowDirection automatically):**

```xml
<TextBlock Text="{x:Bind Description}"
           TextAlignment="Start" />
```
