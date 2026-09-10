---
title: Character Spacing on Uppercase Text
impact: MEDIUM
tags: type, spacing, uppercase, small-caps
---

## Character Spacing on Uppercase Text

Uppercase and small-caps text needs additional letter-spacing for readability. In XAML, CharacterSpacing is in 1/1000th of an em.

**Incorrect (uppercase text feels cramped):**

```xml
<TextBlock Text="SECTION HEADER"
           FontWeight="SemiBold" />
```

**Correct (added character spacing for breathing room):**

```xml
<TextBlock Text="SECTION HEADER"
           FontWeight="SemiBold"
           CharacterSpacing="50" />
```
