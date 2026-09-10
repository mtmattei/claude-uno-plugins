---
title: RepositionThemeTransition for Layout Shifts
impact: HIGH
tags: exit, reposition, layout, reorder
---

## RepositionThemeTransition for Layout Shifts

When sibling elements shift position (e.g., after an item is removed or inserted), RepositionThemeTransition animates remaining items to their new positions instead of snapping.

**Incorrect (remaining items jump after removal):**

```xml
<ItemsRepeater ItemsSource="{x:Bind Items}">
    <ItemsRepeater.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <Border Padding="8">
                <TextBlock Text="{x:Bind Name}" />
            </Border>
        </DataTemplate>
    </ItemsRepeater.ItemTemplate>
</ItemsRepeater>
```

**Correct (siblings slide into new positions):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemContainerTransitions>
        <AddDeleteThemeTransition />
        <RepositionThemeTransition />
    </ListView.ItemContainerTransitions>
</ListView>
```
