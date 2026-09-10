---
title: ConnectedAnimation for Cross-Page Transitions
impact: HIGH
tags: exit, connected-animation, navigation, continuity
---

## ConnectedAnimation for Cross-Page Transitions

ConnectedAnimationService creates fluid transitions between pages by animating a shared element from source to destination. This provides visual continuity during navigation.

**Incorrect (no visual continuity — hard cut between pages):**

```csharp
// SourcePage.xaml.cs
private void OnItemClick(object sender, ItemClickEventArgs e)
{
    Frame.Navigate(typeof(DetailPage), e.ClickedItem);
}
```

**Correct (shared element animates across pages):**

```csharp
// SourcePage.xaml.cs
private void OnItemClick(object sender, ItemClickEventArgs e)
{
    var service = ConnectedAnimationService.GetForCurrentView();
    service.PrepareToAnimate("itemImage", ClickedImage);
    Frame.Navigate(typeof(DetailPage), e.ClickedItem);
}

// DetailPage.xaml.cs
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    var animation = ConnectedAnimationService.GetForCurrentView()
        .GetAnimation("itemImage");
    animation?.TryStart(HeroImage);
}
```
