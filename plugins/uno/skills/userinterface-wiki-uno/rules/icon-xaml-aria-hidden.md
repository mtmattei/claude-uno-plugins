---
title: Mark Decorative Icons as Not Accessible
impact: MEDIUM
tags: icon, accessibility, automation-properties
---

## Mark Decorative Icons as Not Accessible

Decorative icons (next to a label) should be hidden from screen readers with AccessibilityView="Raw". Only set AutomationProperties.Name on icons that are the sole indicator of meaning.

**Incorrect (icon announces redundantly alongside label):**

```xml
<StackPanel Orientation="Horizontal">
    <FontIcon Glyph="&#xE74D;"
              AutomationProperties.Name="Save" />
    <TextBlock Text="Save" />
</StackPanel>
```

**Correct (decorative icon hidden, label carries meaning):**

```xml
<StackPanel Orientation="Horizontal">
    <FontIcon Glyph="&#xE74D;"
              AutomationProperties.AccessibilityView="Raw" />
    <TextBlock Text="Save" />
</StackPanel>
```
