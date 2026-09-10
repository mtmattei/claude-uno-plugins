---
title: Oldstyle Numbers for Prose
impact: MEDIUM
tags: type, numeric, oldstyle, prose
---

## Oldstyle Numbers for Prose

Oldstyle (lowercase) numbers have ascenders and descenders that blend naturally into body text. Use them in paragraphs, not in tables.

**Incorrect (lining numbers look mechanical in prose):**

```xml
<TextBlock Text="Founded in 1984 with 27 employees"
           Typography.NumeralStyle="Lining" />
```

**Correct (oldstyle numbers blend into text):**

```xml
<TextBlock Text="Founded in 1984 with 27 employees"
           Typography.NumeralStyle="OldStyle" />
```
