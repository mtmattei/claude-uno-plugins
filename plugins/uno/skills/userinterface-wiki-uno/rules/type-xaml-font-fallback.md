---
title: Font Fallback Chains
impact: HIGH
tags: type, font, fallback, cross-platform
---

## Font Fallback Chains

On Uno Platform, fonts may not be available on all targets. Always specify a fallback chain so text renders correctly on every platform.

**Incorrect (single font with no fallback — missing on Android/WASM):**

```xml
<TextBlock FontFamily="Segoe UI" Text="Hello" />
```

**Correct (embedded font with platform fallback):**

```xml
<TextBlock FontFamily="/Assets/Fonts/Inter-Regular.ttf#Inter, Segoe UI, Roboto, San Francisco, Helvetica"
           Text="Hello" />
```
