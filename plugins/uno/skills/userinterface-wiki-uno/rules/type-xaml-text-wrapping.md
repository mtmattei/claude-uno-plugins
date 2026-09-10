---
title: Text Wrapping for Multi-Line Content
impact: HIGH
tags: type, wrapping, multiline
---

## Text Wrapping for Multi-Line Content

TextBlock defaults to NoWrap. Body text, descriptions, and any multi-line content must explicitly set TextWrapping="WrapWholeWords" to avoid horizontal overflow.

**Incorrect (text runs off-screen on narrow layouts):**

```xml
<TextBlock Text="{x:Bind Description}" />
```

**Correct (wraps at word boundaries):**

```xml
<TextBlock Text="{x:Bind Description}"
           TextWrapping="WrapWholeWords" />
```
