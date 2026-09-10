---
title: Clip Content During Animated Resize
impact: MEDIUM
tags: layout, clip, resize, overflow
---

## Clip Content During Animated Resize

When animating container size, content can overflow during the transition. Set Clip on the container to prevent visual bleed.

**Incorrect (content overflows during height animation):**

```xml
<Border x:Name="ExpandablePanel" Height="0">
    <StackPanel>
        <TextBlock Text="This leaks out during animation" />
    </StackPanel>
</Border>
```

**Correct (clip constrains content during resize):**

```xml
<Border x:Name="ExpandablePanel" Height="0">
    <Border.Clip>
        <RectangleGeometry Rect="0,0,9999,0"
                           x:Name="PanelClip" />
    </Border.Clip>
    <StackPanel>
        <TextBlock Text="Properly clipped during animation" />
    </StackPanel>
</Border>
```

```csharp
// Animate both Height and Clip rect together
private void Expand(double targetHeight)
{
    PanelClip.Rect = new Rect(0, 0, 9999, targetHeight);
    // Animate ExpandablePanel.Height to targetHeight
}
```
