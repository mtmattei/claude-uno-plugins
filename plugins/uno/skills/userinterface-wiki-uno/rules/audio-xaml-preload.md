---
title: Preload Audio to Avoid First-Play Delay
impact: HIGH
tags: audio, preload, latency
---

## Preload Audio to Avoid First-Play Delay

The first play of a MediaSource has loading latency. Preload sounds during page initialization so they play instantly on interaction.

**Incorrect (loads audio on first click — noticeable delay):**

```csharp
private async void OnButtonClick(object sender, RoutedEventArgs e)
{
    var player = new MediaPlayer();
    player.Source = MediaSource.CreateFromUri(new Uri("ms-appx:///Assets/success.wav"));
    player.Play(); // First play is delayed
}
```

**Correct (preload during init, play instantly later):**

```csharp
private readonly MediaPlayer _successPlayer = new();

protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    _successPlayer.Source = MediaSource.CreateFromUri(
        new Uri("ms-appx:///Assets/success.wav"));
    _successPlayer.Volume = 0.3;
}

private void OnSuccess()
{
    _successPlayer.Position = TimeSpan.Zero;
    _successPlayer.Play(); // Instant
}
```
