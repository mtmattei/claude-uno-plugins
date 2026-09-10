---
title: Always Set FallbackIconSource
impact: HIGH
tags: icon, fallback, animated-icon
---

## Always Set FallbackIconSource

AnimatedIcon may fail to load its visual on some platforms. Always set FallbackIconSource so the control never renders empty.

**Incorrect (no fallback — blank space if animation fails to load):**

```xml
<AnimatedIcon>
    <animatedvisuals:AnimatedSettingsIcon />
</AnimatedIcon>
```

**Correct (FontIcon fallback ensures something always renders):**

```xml
<AnimatedIcon>
    <animatedvisuals:AnimatedSettingsIcon />
    <AnimatedIcon.FallbackIconSource>
        <FontIconSource Glyph="&#xE713;" />
    </AnimatedIcon.FallbackIconSource>
</AnimatedIcon>
```
