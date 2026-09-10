---
title: Provide Toggle to Disable UI Sounds
impact: HIGH
tags: audio, accessibility, settings, toggle
---

## Provide Toggle to Disable UI Sounds

Every sound must have a visual equivalent, and users must be able to disable all UI sounds independently of system volume. Store this preference in app settings.

**Incorrect (no way to disable UI sounds):**

```csharp
private void OnSuccess()
{
    _sfxPlayer.Play(); // Always plays, no opt-out
    ShowSuccessVisual();
}
```

**Correct (check user preference, always show visual):**

```csharp
private void OnSuccess()
{
    if (_settings.UISoundsEnabled)
    {
        _sfxPlayer.Position = TimeSpan.Zero;
        _sfxPlayer.Play();
    }
    ShowSuccessVisual(); // Visual feedback always shows
}
```
