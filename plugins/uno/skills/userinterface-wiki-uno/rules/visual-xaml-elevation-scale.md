---
title: Shadow Matches Elevation in Consistent Scale
impact: MEDIUM
tags: visual, shadow, elevation, z-index, scale
---

## Shadow Matches Elevation in Consistent Scale

Define elevation levels (0, 1, 2, 4, 8, 16, 32) and use consistent Translation.Z values. Arbitrary Z values break the spatial model.

**Incorrect (random Z values with no relationship):**

```xml
<Border Translation="0,0,7">...</Border>   <!-- What level is this? -->
<Border Translation="0,0,23">...</Border>  <!-- Higher than what? -->
<Border Translation="0,0,3">...</Border>   <!-- No pattern -->
```

**Correct (defined elevation levels):**

```xml
<!-- Elevation level 0: flat (no shadow) -->
<Border Translation="0,0,0">...</Border>

<!-- Elevation level 1: subtle (cards, list items) -->
<Border Translation="0,0,4">...</Border>

<!-- Elevation level 2: raised (FAB, app bar) -->
<Border Translation="0,0,8">...</Border>

<!-- Elevation level 3: overlay (dialogs, menus) -->
<Border Translation="0,0,16">...</Border>

<!-- Elevation level 4: top (drawers, navigation) -->
<Border Translation="0,0,32">...</Border>
```
