---
title: Text Trimming for Overflow
impact: HIGH
tags: type, trimming, overflow, ellipsis
---

## Text Trimming for Overflow

Always set TextTrimming on text that can overflow its container. CharacterEllipsis is safest; WordEllipsis looks cleaner but can hide single long words entirely.

**Incorrect (text overflows or clips silently):**

```xml
<TextBlock Text="{x:Bind Title}"
           MaxLines="1" />
```

**Correct (ellipsis signals truncation):**

```xml
<TextBlock Text="{x:Bind Title}"
           MaxLines="1"
           TextTrimming="CharacterEllipsis" />
```
