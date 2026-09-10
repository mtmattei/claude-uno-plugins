---
title: Reuse a Single MediaPlayer Instance
impact: HIGH
tags: audio, media-player, lifecycle, reuse
---

## Reuse a Single MediaPlayer Instance

Creating a new MediaPlayer per sound causes allocation overhead and audible delay. Create one instance and swap the source.

**Incorrect (new MediaPlayer per click — GC pressure and latency):**

```csharp
private void PlayClickSound()
{
    var player = new MediaPlayer();
    player.Source = MediaSource.CreateFromUri(new Uri("ms-appx:///Assets/click.wav"));
    player.Play();
}
```

**Correct (reuse single player, reset position):**

```csharp
private readonly MediaPlayer _sfxPlayer = new();

private void PlayClickSound()
{
    _sfxPlayer.Source = MediaSource.CreateFromUri(new Uri("ms-appx:///Assets/click.wav"));
    _sfxPlayer.Position = TimeSpan.Zero;
    _sfxPlayer.Play();
}

// Dispose in page cleanup
public void Dispose() => _sfxPlayer.Dispose();
```
