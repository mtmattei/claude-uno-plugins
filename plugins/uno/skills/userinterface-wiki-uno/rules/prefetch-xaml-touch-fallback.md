---
title: Touch Fallback for Pointer Prediction
impact: MEDIUM
tags: prefetch, touch, mobile, fallback
---

## Touch Fallback for Pointer Prediction

Pointer trajectory prediction requires a cursor. On touch devices, fall back to focus-based or tap-hint-based prefetching since there's no hover state.

**Incorrect (trajectory prediction fails silently on touch):**

```csharp
private void OnPointerMoved(object sender, PointerRoutedEventArgs e)
{
    // This never fires on tap-only devices
    PredictAndPrefetch(e.GetCurrentPoint(this).Position);
}
```

**Correct (check input type and use appropriate strategy):**

```csharp
private void OnPointerMoved(object sender, PointerRoutedEventArgs e)
{
    if (e.Pointer.PointerDeviceType == PointerDeviceType.Mouse)
    {
        PredictAndPrefetch(e.GetCurrentPoint(this).Position);
    }
    // Touch users get focus-based prefetch via GotFocus handlers
}
```
