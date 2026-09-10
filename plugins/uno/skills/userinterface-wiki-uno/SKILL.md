---
name: userinterface-wiki-uno
description: UI/UX best practices adapted for WinUI 3, XAML, and Uno Platform. Use when reviewing animations, visual states, typography, layout animation, audio, icons, prefetching, or visual design in XAML/C# apps. Covers 8 categories. Outputs file:line findings. Do NOT use as the orchestrator for a broad production-quality pass over an existing design — that is gold-standard-pass, which invokes this skill for per-detail correctness.
license: MIT
metadata:
  author: adapted from raphael-salaja/userinterface-wiki
  version: "1.0.0"
  category: "ui-ux"
---

# User Interface Wiki — Uno Platform / WinUI 3 Adaptation

UI/UX best practices adapted from the web-focused userinterface-wiki for WinUI 3, XAML, and Uno Platform cross-platform applications. The platform-neutral foundations (Animation Principles, Timing Functions, Laws of UX) live in `rules/universal-principles.md` in this skill — the original web-focused skill has been retired. Everything else here is the platform-specific adaptation layer.

## When to Apply

Reference these guidelines when:
- Implementing exit/entrance transitions with ThemeTransition or Storyboard
- Using ConnectedAnimationService for cross-page transitions
- Managing visual states with VisualStateManager
- Working with XAML Typography attached properties (NumeralAlignment, NumeralStyle, etc.)
- Animating layout changes with composition implicit animations
- Building animated icons with AnimatedIcon or Lottie/AnimatedVisualPlayer
- Adding audio feedback with MediaPlayer or AudioGraph
- Applying visual polish (ThemeShadow, CornerRadius, spacing scales)
- Prefetching data based on pointer trajectory or focus patterns

## Rule Categories by Priority

| Priority | Category | Impact | Prefixes |
|----------|----------|--------|----------|
| 1 | XAML Typography | HIGH | `type-xaml-` |
| 2 | Transitions & Exit Animations | HIGH | `exit-xaml-` |
| 3 | Visual States & Overlays | HIGH | `state-xaml-` |
| 4 | Layout Animation | MEDIUM | `layout-xaml-` |
| 5 | Data Prefetching | MEDIUM | `prefetch-xaml-` |
| 6 | Animated Icons | MEDIUM | `icon-xaml-` |
| 7 | XAML Audio | MEDIUM | `audio-xaml-` |
| 8 | Visual Design XAML | HIGH | `visual-xaml-` |

## How to Use

Read `rules/universal-principles.md` first for the platform-neutral foundations (animation principles, timing functions, Laws of UX), then individual rule files for detailed explanations and code examples:

```
rules/universal-principles.md
rules/type-xaml-tabular-nums.md
rules/exit-xaml-theme-transition.md
rules/state-xaml-use-vsm.md
rules/visual-xaml-concentric-radius.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
