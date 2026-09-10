---
title: Default Volume Should Be Subtle
impact: MEDIUM
tags: audio, volume, subtle
---

## Default Volume Should Be Subtle

UI sound effects should be background confirmation, not foreground noise. Default to 0.3 volume, not 1.0.

**Incorrect (full volume — startles user):**

```csharp
_sfxPlayer.Volume = 1.0;
_sfxPlayer.Play();
```

**Correct (subtle default — confirms without annoying):**

```csharp
_sfxPlayer.Volume = 0.3;
_sfxPlayer.Play();
```
