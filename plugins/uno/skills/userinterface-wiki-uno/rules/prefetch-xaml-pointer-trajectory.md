---
title: Pointer Trajectory Prediction Over PointerEntered
impact: HIGH
tags: prefetch, pointer, trajectory, navigation
---

## Pointer Trajectory Prediction Over PointerEntered

PointerEntered fires only when the cursor reaches the element. Track PointerMoved on a parent and predict which child the cursor is heading toward to start prefetching 100-200ms earlier.

**Incorrect (prefetches on hover — too late):**

```csharp
private void OnItemPointerEntered(object sender, PointerRoutedEventArgs e)
{
    if (sender is FrameworkElement { DataContext: NavigationItem item })
    {
        _ = PrefetchService.LoadAsync(item.Route);
    }
}
```

**Correct (trajectory prediction starts prefetch earlier):**

```csharp
private Point _lastPosition;

private void OnPanelPointerMoved(object sender, PointerRoutedEventArgs e)
{
    var current = e.GetCurrentPoint(NavigationPanel).Position;
    var velocity = new Point(current.X - _lastPosition.X, current.Y - _lastPosition.Y);
    _lastPosition = current;

    var predicted = new Point(current.X + velocity.X * 5, current.Y + velocity.Y * 5);
    var target = VisualTreeHelper.FindElementsInHostCoordinates(predicted, NavigationPanel)
        .OfType<FrameworkElement>()
        .FirstOrDefault(e => e.DataContext is NavigationItem);

    if (target?.DataContext is NavigationItem item)
    {
        _ = PrefetchService.LoadAsync(item.Route);
    }
}
```
