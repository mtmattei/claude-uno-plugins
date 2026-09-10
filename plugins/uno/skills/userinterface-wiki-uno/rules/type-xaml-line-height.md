---
title: Explicit Line Height for Consistent Rhythm
impact: MEDIUM
tags: type, line-height, rhythm, spacing
---

## Explicit Line Height for Consistent Rhythm

Default line height varies across platforms. Set LineHeight explicitly on body text to ensure vertical rhythm is consistent across Windows, Android, iOS, and WASM.

**Incorrect (platform-dependent line height — layout shifts between targets):**

```xml
<TextBlock Text="{x:Bind Body}"
           FontSize="14" />
```

**Correct (explicit line height — consistent on all platforms):**

```xml
<TextBlock Text="{x:Bind Body}"
           FontSize="14"
           LineHeight="20"
           LineStackingStrategy="BlockLineHeight" />
```
