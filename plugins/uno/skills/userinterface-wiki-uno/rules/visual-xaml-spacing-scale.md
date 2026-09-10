---
title: Use a Consistent Spacing Scale
impact: HIGH
tags: visual, spacing, scale, consistency
---

## Use a Consistent Spacing Scale

Define spacing as resource doubles (4, 8, 12, 16, 24, 32, 48) and reference them throughout. Arbitrary margin/padding values create visual inconsistency.

**Incorrect (arbitrary values — 7px here, 13px there):**

```xml
<StackPanel Margin="7,13,7,5">
    <TextBlock Margin="0,0,0,9" Text="Title" />
    <TextBlock Margin="0,0,0,11" Text="Subtitle" />
</StackPanel>
```

**Correct (consistent scale from resources):**

```xml
<!-- In ResourceDictionary -->
<x:Double x:Key="SpacingXS">4</x:Double>
<x:Double x:Key="SpacingSM">8</x:Double>
<x:Double x:Key="SpacingMD">12</x:Double>
<x:Double x:Key="SpacingLG">16</x:Double>
<x:Double x:Key="SpacingXL">24</x:Double>

<!-- Usage -->
<StackPanel Margin="{StaticResource SpacingLG}">
    <TextBlock Margin="0,0,0,{StaticResource SpacingSM}" Text="Title" />
    <TextBlock Margin="0,0,0,{StaticResource SpacingSM}" Text="Subtitle" />
</StackPanel>
```
