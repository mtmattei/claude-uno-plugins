---
title: AnimatedVisualPlayer for Complex Icon Animation
impact: MEDIUM
tags: icon, lottie, animated-visual-player
---

## AnimatedVisualPlayer for Complex Icon Animation

For custom Lottie animations beyond built-in AnimatedIcon, use AnimatedVisualPlayer with explicit playback control. Set AutoPlay="False" to control when segments play.

**Incorrect (AutoPlay runs animation on load — wasted motion):**

```xml
<AnimatedVisualPlayer x:Name="SuccessAnimation"
                      AutoPlay="True">
    <lottie:LottieVisualSource UriSource="ms-appx:///Assets/success.json" />
</AnimatedVisualPlayer>
```

**Correct (play on demand at the right moment):**

```xml
<AnimatedVisualPlayer x:Name="SuccessAnimation"
                      AutoPlay="False">
    <lottie:LottieVisualSource UriSource="ms-appx:///Assets/success.json" />
</AnimatedVisualPlayer>
```

```csharp
private async Task ShowSuccess()
{
    SuccessAnimation.Visibility = Visibility.Visible;
    await SuccessAnimation.PlayAsync(0, 1, looped: false);
}
```
