---
name: xaml-design-polish
description: The motion-and-feel system for Uno Platform / WinUI / XAML — the tuned numbers and taste that separate "an LLM built this" from "a human with taste built this." House easing curves (KeySpline/Composition), token discipline, drag physics & springs, two-zone snap points, blur-in entrances, layered shadows, tactile press, height reveals, reduced-motion, and state-driven design. Use when building or reviewing motion, micro-interactions, component states, entrances, or the feel of an interaction in XAML/C#. Pairs with userinterface-wiki-uno (the XAML correctness rule reference) — this skill owns how it FEELS, that one owns whether each detail is right. Lead every motion request with exact numbers, never adjectives. Do NOT use as the orchestrator for a broad production-quality pass over an existing design — that is gold-standard-pass, which invokes this skill for its numbers.
---

# Design Polish: motion & feel for Uno / WinUI / XAML

A working system for UI that feels hand-built, expressed in XAML / C# / Composition. The throughline: **give numbers, never adjectives.** "Smooth" is unbuildable; `KeySpline="0.22,1 0.36,1"` at 280ms is. Define the feel once as tokens, then every component inherits it.

## Role split — read this first

This skill is the **feel layer**. It does not re-document XAML correctness rules that already live in `userinterface-wiki-uno`. Route by intent:

| You are… | Use |
|---|---|
| Designing how a motion/interaction should *feel*, picking curves/durations, building drag physics, snap, entrances, the token system | **xaml-design-polish** (this skill) |
| Checking a XAML detail is *correct* — VSM state sets, exit transitions, typography props, concentric radius, spacing scale, hit areas, ShadowContainer/ThemeShadow, animated icons, prefetch | **userinterface-wiki-uno** |

When this skill says "defer to userinterface-wiki-uno," go read that rule there rather than duplicating it here. House values pinned below (curves, durations, `0.98` press) are authoritative across both.

## How to apply

1. **Drop in the token system first** (`MotionTokens.xaml`, same folder — merge it in `App.xaml`). Forbid one-off curves, durations, radii. This kills most of the "generated UI" look before a single control exists.
2. **Specify motion as exact KeySpline points + durations + offsets**, never feelings. When you must describe a feel, anchor to a reference ("like an iOS sheet: weighty, slightly springy, settles fast").
3. **Prefer Composition over Storyboard** for offset/scale/opacity/blur — it runs off the UI thread (validated: `userinterface-wiki-uno` → *Composition Animations Over Storyboard for Layout*). Reserve Storyboards/ThemeTransitions for standard one-shot enter/exit.
4. **Think in states, then list them.** A control is a system (Normal / PointerOver / Pressed / Disabled / Focused + domain states like loading/working/success), not a picture.
5. **Isolate when iterating.** "Now only tune the shadow stack." One variable at a time.

## House token system → `MotionTokens.xaml`

The canonical artifact is `MotionTokens.xaml` (this folder): easing `KeySpline`s, radius doubles, durations (both `Duration` objects and `*Ms` doubles for Composition), entrance/press constants, and Toolkit `ShadowCollection` stacks. Merge it:

```xml
<ResourceDictionary Source="ms-appx:///Themes/MotionTokens.xaml" />
```

The three house curves (the two control points *are* the cubic-bezier handles; mirror them in Composition with `CreateCubicBezierEasingFunction` where supported):

| Token | KeySpline | Composition Vector2 pair | Use |
|---|---|---|---|
| `EaseSmooth` | `0.22,1 0.36,1` | `(0.22,1) (0.36,1)` | default for almost everything |
| `EaseOut` | `0.17,1 0.32,1` | `(0.17,1) (0.32,1)` | decorative entrances |
| `EaseInOut` | `0.66,0 0.34,1` | `(0.66,0) (0.34,1)` | symmetric moves |

**There is no `EaseSpring` easing curve.** A `KeySpline`'s control points are clamped to `[0,1]` on both axes (unlike CSS `cubic-bezier`), so an easing curve *cannot* overshoot. A pop/bounce comes from overshooting the animated **value** — a scale sequence `1 → 1.05 → 1` (rise on `EaseOut`, settle on `EaseSmooth`) — or from a Composition spring (`DampingRatio ~0.7`, `Period ~80ms`) where the target supports it. Never encode a bounce in the curve.

Durations: fast 150ms, normal 200ms, slow 280ms. Radius: 6 / 12 / 24. These numbers are tuned by feel — a curve 0.02 off feels subtly wrong. Keep using Material color/type-scale resources for colour and fonts; these tokens are additive (easing, duration, radius, shadow). For spacing scale, defer to `userinterface-wiki-uno` → *Use a Consistent Spacing Scale*.

## The ten rules (XAML/C#-native)

### 1. Easing is everything. The default ease is banned.
The biggest tell of generated UI is the easing curve. Never the system default `VisualTransition`/`ThemeTransition` curve for anything expressive. Drive Storyboard spline keyframes from the house `KeySpline`s, or Composition animations from `CreateCubicBezierEasingFunction`.
**Build it:**
```xml
<SplineDoubleKeyFrame KeyTime="0:0:0.28" Value="0" KeySpline="{StaticResource EaseSmooth}" />
```
**Prompt it:** "Use `KeySpline 0.22,1 0.36,1` for transitions. For a badge that pops in with a tiny bounce, overshoot the *scale* — keyframe `1 → 1.05 → 1`, rising on `EaseOut` and settling on `EaseSmooth` — not the curve, since a KeySpline can't overshoot."

### 2. Define tokens before building a single control.
Polish reads as consistency; consistency comes from a shared vocabulary. Tokens stop the model inventing one-off `13px` radii and random `0.3s` timings.
**Prompt it:** Hand the model `MotionTokens.xaml` first: "use only these resources, no one-off values." (Spacing/radius correctness → `userinterface-wiki-uno`.)

### 3. For anything draggable, use real physics, not fades and slides.
Three things make a drag feel alive: (a) velocity tracking from `ManipulationDelta` so a flick has weight, (b) momentum on release via inertia that coasts to rest, (c) soft boundaries that stretch and spring back at the edge. For un-tween-able values (counters, live numbers) use a **spring**, not a fixed duration.
**Build it:** `Compositor.CreateSpringVector3Animation` / `CreateSpringScalarAnimation` (`DampingRatio ~0.7`, `Period ~80ms`) — validated in `userinterface-wiki-uno` → *Use Spring NaturalMotion for Organic Layout*. Manipulation events (`ManipulationMode`, `ManipulationDelta.Velocities`) give velocity; inertia gives momentum.
**Prompt it (feel + reference):** "Make the slider feel like a real physical object. When I flick it, it keeps gliding and coasts to a stop on its own, like sliding something across a table. At the edge it stretches a little and springs back."

### 4. Add snap points. Magnetic snapping is free haptics.
As the handle nears a meaningful value it magnetizes to it. The trick is a **two-zone system**: a tight pull-in zone to snap, a larger release zone to break free — so it locks and resists. Pulse the label when it catches (a quick `VisualState` / Composition scale-bump).
**Build it:** snap math in `ManipulationDelta`; settle with a spring into the snapped value; for `ScrollViewer`-based UIs, built-in snap points cover the simple case.
**Prompt it:** "Magnetic snap points at [values]. Smaller pull-in zone, larger release zone so it locks and resists, and flash the label when it catches."

### 5. Entrances blur in. They never just fade.
Combine three things on `Loaded`: `Opacity 0→1`, `Translation.Y 6→0`, and a Composition `GaussianBlur 2px→0`, ~280ms on `EaseSmooth`. The blur makes content *focus into place* instead of flicking on. For standard, non-hero entrances `EntranceThemeTransition` is fine (defer to `userinterface-wiki-uno`); the blur recipe is the premium version.
**Build it:** Composition `ScalarKeyFrameAnimation` (opacity) + `Vector3KeyFrameAnimation` (`Translation`) sharing the house easing. The GaussianBlur step is the harder part on Uno: the `GaussianBlurEffect` exists but its Win2D wrapper is still internal, so you hand-implement `IGraphicsEffectD2D1Interop` (Uno's `composition.md` ships the boilerplate), and Composition effects can fail to render on software/CPU rendering. Treat the blur as the optional hero touch with a clean fade+rise fallback when it isn't available.
**Prompt it:** "Entrance = fade + 6px rise + a 2px Composition blur that clears, ~280ms on EaseSmooth."

### 6. One shadow is a sticker. Real depth is layered light.
Physical objects cast several shadows at once. Three rules: (a) a **hairline ring** replaces the border — the single biggest tell of hand-made UI; the edge is light, not a 1px stroke. (b) Opacities stay tiny (~2–8%); never pure black (validated: `userinterface-wiki-uno` → *No Pure Black Shadows*). (c) Stack several blurs at different sizes (tight contact + wide ambient).
**Build it:** Uno Toolkit `ShadowContainer` with multiple `Shadow` entries — `MotionTokens.xaml` ships `CardShadows` and `ElevatedShadows` collections. Full ShadowContainer/ThemeShadow/elevation-scale guidance → `userinterface-wiki-uno` → *Use Uno Toolkit ShadowContainer*, *Shadow Matches Elevation*. Toolkit `Blur` renders softer than CSS blur radius — tune to taste.
**Prompt it:** "Don't use one drop shadow. ShadowContainer with a hairline ring instead of a border, a tight contact shadow, and a wide soft ambient, all at 2–8% opacity; animate the whole stack on hover."

### 7. Make everything tactile. Press should be felt.
Every interactive element gets a press response: scale to **`0.98`** (firm press, not a collapse) over `DurationFast`. This is the pinned house value — it already matches `userinterface-wiki-uno`'s Pressed-state example; `0.96`/`0.97` from other sources are superseded.
**Build it:** a `Pressed` `VisualState` driving a `ScaleTransform` to `0.98`, or Composition scale on `PointerPressed`/`PointerReleased`. Full state-set requirements → `userinterface-wiki-uno` → *Define All Interactive States*.
**Prompt it:** "Every clickable element scales to 98% when pressed, fast duration. Tooltips fade + lift 4px + clear a 2px blur, never pop in."

### 8. Reveal height the right way. No fake tricks.
Animate the measured height with the **measure/animate separation** (inner element measures, outer animates — never the same element, or you get a feedback loop) and **clip during resize** so content doesn't bleed. Both validated in `userinterface-wiki-uno` → *Separate Measurement from Animation Target*, *Clip Content During Animated Resize*. For an element moving *across* the layout (a card flying into another container), use `ConnectedAnimationService`, or a manual FLIP: capture `TransformToVisual` before and after, apply an inverse Composition offset, animate to identity.
**Prompt it:** "Expand/collapse via measure-on-inner / animate-on-wrapper with Clip; card-between-containers via ConnectedAnimation."

### 9. Respect performance and accessibility, or it's not polished.
Gate decorative motion on `UISettings().AnimationsEnabled` — when off, jump to the end state and stop loops (validated: `userinterface-wiki-uno` → *Respect Reduced Motion*). For 60fps, animate `Translation` / `Opacity` / `Scale` (Composition, off-thread) over layout/height/shadow; never run implicit animations on virtualized panels (use `ItemContainerTransitions` instead). Reserve heavy effects (blur, shadow stacks) for small deliberate moments.
**Prompt it:** "Honor reduced motion everywhere. Favor Translation/Opacity/Scale on long lists; use ItemContainerTransitions, not per-item implicit animations."

### 10. State-driven design is the actual job.
The most important rule. A control is a system of states, and **you won't know all the states until you build it.** The mockup always looks complete; then you drag the thing and feel the holes — "this needs a working state," "the number should roll, not swap," "this label should shimmer while busy," "the icon should cross-fade between play and pause." Those micro-interactions are *discovered through use, never specced up front*.
**Build it:** full `VisualStateManager` set + `VisualTransition` for smooth changes; icon morphs via `AnimatedIcon` (cross-fade, not glyph swap); roll/shimmer via Composition. Defer to `userinterface-wiki-uno` → *Use VisualStateManager*, *Animate Between Visual States*, *Use AnimatedIcon*.
**Prompt it (working state):** "While it's working, no spinner. Let the label text glow softly, like light sweeping across the word side to side and back, looping ~2s. Calm and alive, not flashy."

## Cross-platform note (Uno)

`ElementCompositionPreview` and `Translation`/`Opacity`/`Scale` Composition animations are well-supported on Uno Skia and are the portable core. **Verify before relying on the rest**: Uno's official Composition *Implemented APIs* list (`composition.md`) enumerates `Compositor`, `CompositionAnimation`, and `ExpressionAnimation` but does **not** list the spring (`SpringScalarNaturalMotionAnimation`), keyframe (`ScalarKeyFrameAnimation`/`Vector3KeyFrameAnimation`), or `CubicBezierEasingFunction` types this skill leans on — so do not assume they exist on every target. Storyboard `SplineDoubleKeyFrame` (driven by the house `KeySpline`s) is the guaranteed-portable fallback for tweenable values. Other WinUI niceties also vary: `ThemeShadow`, `ConnectedAnimationService`, GaussianBlur (above), and `AnimatedIcon`/Lottie. Per your scaffolding rule, POC the hero effects on every targeted platform first, and keep a `UISettings.AnimationsEnabled`-style graceful fallback.

## Prompting checklist

- Give numbers, never adjectives (KeySpline points + ms + px offsets).
- Lead with `MotionTokens.xaml`; forbid one-off values.
- Think in states, then list them — the model builds exactly the states you name.
- Isolate when iterating — one variable at a time.
- Describe feeling + a reference ("iOS sheet: weighty, slightly springy, settles fast").
- For correctness of any specific XAML detail, point at `userinterface-wiki-uno`.

## Pinned house values (authoritative)

- Easing: the three `KeySpline`s above (`EaseSmooth`/`EaseOut`/`EaseInOut`). Default = `EaseSmooth`. Overshoot is a value effect, not a curve — there is no `EaseSpring`.
- Duration: fast 150 / normal 200 / slow 280 ms.
- Radius: 6 / 12 / 24.
- Press scale: **0.98**. Shadow opacities: 2–8%, never pure black.

## The part that doesn't come from the model

The model is the hands; the eye is yours. It will build a flawless spring or a ConnectedAnimation faster than anyone by hand — but it didn't decide the press should be `0.98` not `0.95`, that the entrance needed a 2px blur, that the release zone should be bigger than the pull-in zone, or that the control was missing a working state until you felt the gap. That judgment is taste, earned from shipping. Carry Rule 10 as the mindset into all nine: the mockup tells you where to start; the build tells you what it actually needs.
