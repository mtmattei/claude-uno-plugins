---
title: Tabular Numbers for Data Display
impact: HIGH
tags: type, numeric, tabular, alignment
---

## Tabular Numbers for Data Display

Use Typography.NumeralAlignment="Tabular" for any numeric data that should align in columns (tables, dashboards, pricing). Tabular figures are monospaced so digits stack vertically.

**Incorrect (proportional numbers misalign in columns):**

```xml
<TextBlock Text="{x:Bind Price}" />
```

**Correct (tabular numbers align):**

```xml
<TextBlock Text="{x:Bind Price}"
           Typography.NumeralAlignment="Tabular" />
```
