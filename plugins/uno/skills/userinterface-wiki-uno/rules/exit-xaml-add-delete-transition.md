---
title: AddDeleteThemeTransition for Collection Changes
impact: HIGH
tags: exit, collection, add, delete, list
---

## AddDeleteThemeTransition for Collection Changes

When items are added to or removed from a collection, use AddDeleteThemeTransition so the list animates smoothly instead of snapping.

**Incorrect (list items appear/disappear instantly):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <TextBlock Text="{x:Bind Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**Correct (items animate in and out):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemContainerTransitions>
        <AddDeleteThemeTransition />
    </ListView.ItemContainerTransitions>
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <TextBlock Text="{x:Bind Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```
