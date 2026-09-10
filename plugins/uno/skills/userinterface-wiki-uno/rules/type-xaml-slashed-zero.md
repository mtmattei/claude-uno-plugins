---
title: Slashed Zero for Code-Adjacent UI
impact: MEDIUM
tags: type, numeric, slashed-zero, disambiguation
---

## Slashed Zero for Code-Adjacent UI

In code-adjacent contexts (IDs, serial numbers, hex values), a slashed zero prevents confusion with the letter O.

**Incorrect (zero and O are indistinguishable):**

```xml
<TextBlock Text="Order #O01230"
           FontFamily="Consolas" />
```

**Correct (slashed zero disambiguates):**

```xml
<TextBlock Text="Order #O01230"
           FontFamily="Consolas"
           Typography.SlashedZero="True" />
```
