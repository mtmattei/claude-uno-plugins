---
title: Disable Interactions on Exiting Elements
impact: HIGH
tags: exit, interaction, hit-test, pointer
---

## Disable Interactions on Exiting Elements

Elements playing an exit animation are still in the visual tree and can receive pointer events. Disable hit-testing to prevent ghost clicks during exit.

**Incorrect (fading element still receives taps):**

```csharp
private void RemoveCard(UIElement card)
{
    var sb = CreateFadeOutStoryboard(card);
    sb.Completed += (_, _) => Panel.Children.Remove(card);
    sb.Begin();
}
```

**Correct (disable hit-testing immediately):**

```csharp
private void RemoveCard(UIElement card)
{
    card.IsHitTestVisible = false;
    var sb = CreateFadeOutStoryboard(card);
    sb.Completed += (_, _) => Panel.Children.Remove(card);
    sb.Begin();
}
```
