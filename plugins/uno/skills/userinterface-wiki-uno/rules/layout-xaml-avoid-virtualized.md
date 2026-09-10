---
title: Don't Animate Layout on Virtualized Panels
impact: HIGH
tags: layout, virtualization, performance, listview
---

## Don't Animate Layout on Virtualized Panels

Virtualized panels (ListView, GridView, ItemsRepeater with virtualizing layout) recycle containers. Implicit layout animations on recycled containers cause ghost animations and visual glitches.

**Incorrect (implicit offset animation on ListView items — recycled items ghost-slide):**

```csharp
private void OnContainerContentChanging(ListViewBase sender,
    ContainerContentChangingEventArgs args)
{
    var visual = ElementCompositionPreview.GetElementVisual(args.ItemContainer);
    var group = visual.Compositor.CreateImplicitAnimationCollection();
    group["Offset"] = CreateSpringAnimation(visual.Compositor);
    visual.ImplicitAnimations = group; // Bad on recycled containers
}
```

**Correct (use built-in ItemContainerTransitions instead):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemContainerTransitions>
        <AddDeleteThemeTransition />
        <RepositionThemeTransition />
    </ListView.ItemContainerTransitions>
</ListView>
```
