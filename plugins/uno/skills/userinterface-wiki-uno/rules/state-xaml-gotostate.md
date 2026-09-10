---
title: Use GoToState Not Direct Property Changes
impact: HIGH
tags: state, gotostate, vsm, code-behind
---

## Use GoToState Not Direct Property Changes

When you must trigger state changes from code, use VisualStateManager.GoToState rather than setting properties directly. GoToState respects transitions and keeps state consistent.

**Incorrect (direct property set — bypasses VSM transitions):**

```csharp
private void OnError()
{
    ErrorBorder.BorderBrush = new SolidColorBrush(Colors.Red);
    ErrorIcon.Visibility = Visibility.Visible;
}
```

**Correct (GoToState triggers defined visual state with transitions):**

```csharp
private void OnError()
{
    VisualStateManager.GoToState(this, "ErrorState", useTransitions: true);
}
```
