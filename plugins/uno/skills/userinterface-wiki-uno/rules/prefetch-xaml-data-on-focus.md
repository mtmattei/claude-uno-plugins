---
title: Prefetch Data on Focus Before Selection
impact: HIGH
tags: prefetch, focus, keyboard, navigation
---

## Prefetch Data on Focus Before Selection

When a user navigates a list with keyboard, the focused item will likely be selected next. Start loading data on GotFocus rather than waiting for SelectionChanged.

**Incorrect (loads data only after selection — keyboard users wait):**

```csharp
private async void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (e.AddedItems.FirstOrDefault() is Category category)
    {
        DetailPanel.Content = await LoadCategoryDetails(category.Id);
    }
}
```

**Correct (prefetch on focus, confirm on selection):**

```csharp
private Task<CategoryDetail>? _prefetchTask;

private void OnItemGotFocus(object sender, RoutedEventArgs e)
{
    if (sender is FrameworkElement { DataContext: Category category })
    {
        _prefetchTask = LoadCategoryDetails(category.Id);
    }
}

private async void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (_prefetchTask is not null)
    {
        DetailPanel.Content = await _prefetchTask;
    }
}
```
