---
title: Use AnimatedIcon for State Transitions
impact: HIGH
tags: icon, animated-icon, state, lottie
---

## Use AnimatedIcon for State Transitions

AnimatedIcon integrates with control states (PointerOver, Pressed) to play Lottie segments automatically. Use it instead of swapping between static icons on state change.

**Incorrect (swapping icons on state change — no transition):**

```xml
<Button>
    <FontIcon x:Name="PlayIcon" Glyph="&#xE768;" />
</Button>
```

```csharp
private void OnToggle()
{
    PlayIcon.Glyph = _isPlaying ? "\xE768" : "\xE769"; // Hard swap
}
```

**Correct (AnimatedIcon morphs between states):**

```xml
<Button>
    <AnimatedIcon x:Name="PlayPauseIcon">
        <animatedvisuals:AnimatedPlayPauseIcon />
        <AnimatedIcon.FallbackIconSource>
            <FontIconSource Glyph="&#xE768;" />
        </AnimatedIcon.FallbackIconSource>
    </AnimatedIcon>
</Button>
```
