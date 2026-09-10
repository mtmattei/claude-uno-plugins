---
title: Prefetch by Intent Not Viewport
impact: MEDIUM
tags: prefetch, bandwidth, selective
---

## Prefetch by Intent Not Viewport

Don't prefetch every visible navigation target. Prefetch based on user intent signals (trajectory, focus, frequency) to avoid wasted bandwidth and API calls.

**Incorrect (prefetches all visible items on load — wasteful):**

```csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    foreach (var item in NavigationItems)
    {
        _ = PrefetchService.LoadAsync(item.Route); // 20 API calls at once
    }
}
```

**Correct (prefetch only the most likely target):**

```csharp
private void OnNavigationItemGotFocus(object sender, RoutedEventArgs e)
{
    if (sender is FrameworkElement { DataContext: NavigationItem item })
    {
        _ = PrefetchService.LoadIfNotCachedAsync(item.Route);
    }
}
```
