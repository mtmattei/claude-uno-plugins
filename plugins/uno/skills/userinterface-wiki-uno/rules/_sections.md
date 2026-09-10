# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. XAML Typography (type-xaml)

**Impact:** HIGH
**Description:** XAML Typography attached properties and text formatting that most developers overlook. Numeric alignment, numeral styles, slashed zeros, character spacing, text trimming, and font configuration that make typography feel considered in WinUI/Uno apps.

## 2. Transitions & Exit Animations (exit-xaml)

**Impact:** HIGH
**Description:** ThemeTransition, Storyboard exit patterns, and ConnectedAnimationService for cross-page transitions. Correct usage prevents layout jumps, stale interactions, and jarring removals in XAML apps.

## 3. Visual States & Overlays (state-xaml)

**Impact:** HIGH
**Description:** VisualStateManager patterns, template parts, and overlay techniques that replace CSS pseudo-elements and state-driven styling in XAML. Proper VSM usage ensures consistent interactive feedback across controls.

## 4. Layout Animation (layout-xaml)

**Impact:** MEDIUM
**Description:** Composition implicit animations, LayoutTransition, and SizeChanged-driven animation patterns. Separating measurement from animation to avoid feedback loops in XAML layout changes.

## 5. Data Prefetching (prefetch-xaml)

**Impact:** MEDIUM
**Description:** Loading data before the user navigates by analyzing pointer trajectory, focus patterns, and incremental loading. Reduces perceived latency in Uno/WinUI apps through anticipatory data fetching.

## 6. Animated Icons (icon-xaml)

**Impact:** MEDIUM
**Description:** AnimatedIcon, Lottie via AnimatedVisualPlayer, and animated path transitions. Building icon components that transition between states with platform-native animation APIs.

## 7. XAML Audio (audio-xaml)

**Impact:** MEDIUM
**Description:** MediaPlayer for UI sound effects and AudioGraph for procedural audio in WinUI/Uno apps. Covers lifecycle management, accessibility, and platform-aware audio patterns.

## 8. Visual Design XAML (visual-xaml)

**Impact:** HIGH
**Description:** ThemeShadow, CornerRadius nesting, spacing scales, and brush patterns in XAML. Small details that separate considered interfaces from default ones in WinUI/Uno applications.
