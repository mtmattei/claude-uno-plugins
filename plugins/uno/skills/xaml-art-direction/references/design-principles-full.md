# XAML Design Principles (adapted from web frontend-design)

Load this file when the task is **aesthetic direction and avoiding generic "AI slop"** in WinUI 3 / Uno Platform XAML — i.e. the design-taste layer, not motion mechanics or XAML correctness.

**Role split (do not duplicate):**
- *How motion/interaction should feel* — curves, durations, press, entrances, drag, snap, tokens → `xaml-design-polish` (house numbers are authoritative there).
- *Whether each XAML detail is correct* — states, focus, virtualization, reduced motion, spacing scale → `userinterface-wiki-uno`.
- *Material color roles, type ramp, control styles* → `uno-material`.
- *This file* — bold aesthetic direction + the fingerprints of templated XAML to avoid, translated from the web skill. It is additive to the three above; on conflict, the skills win.

The source web skill enforces most of its rules through CSS features XAML lacks (`clamp`, `oklch`, `@container`, transform-only animation). A literal port is wrong. Every rule below is translated to the XAML **mechanism**, or dropped where the platform already prevents the problem.

---

## Concept translation table

| Web primitive | XAML equivalent | Notes |
|---|---|---|
| `clamp()` fluid type/space | Material type-ramp resources + `ResponsiveExtension` per-breakpoint values | No viewport-relative sizing. Discrete breakpoints, not continuous. |
| Modular type scale | Material type ramp (`DisplayLarge`…`LabelSmall`) via `Style`/`ThemeResource` | Never set `FontSize` literally. |
| `oklch()` / `color-mix()` | Material MD3 color roles (`Primary`, `Surface`, `OnSurfaceVariant`…) via `ThemeResource` | Perceptual palette lives in the Material tonal system, not the markup. |
| `light-dark()` | `ThemeDictionaries` (`Light` / `Dark` / `HighContrast`) | Same key resolves per theme. |
| Custom font (`@font-face`) | `FontFamily` resource → `ms-appx:///Assets/Fonts/…#Family` | Ship **static** weight files (see variable-font gotcha). |
| `transform` / `opacity` animation | `RenderTransform` (`TranslateTransform`/`ScaleTransform`) + `Opacity`, or Composition `Translation`/`Scale`/`Opacity` | Same rule: never animate layout. |
| `grid-template-rows` height reveal | `xaml-design-polish` height-reveal recipe | No CSS-grid row tween; use the skill's pattern. |
| `@container` query | `ResponsiveExtension` (swap values) / `ResponsiveView` (swap templates) / `AdaptiveTrigger` + VSM | Component- and page-level adaptation. |
| `prefers-reduced-motion` | `UISettings().AnimationsEnabled` gate | Per `xaml-design-polish` / `userinterface-wiki-uno`. |
| Modal `<dialog>` | `ContentDialog` | Same "lazy" warning applies. |
| Ghost / text / secondary button | Material `Outlined` / `Text` / `Elevated` / `Filled` button styles | Hierarchy via style, not custom brushes. |

---

## Aesthetic direction (unchanged in spirit)

Commit to one **bold, intentional** direction and execute with precision — refined-minimal and maximalist both work; the failure mode is timid defaults, not low intensity. State the direction before building: purpose, tone (pick an extreme), constraints (targets/platforms/perf), and the one memorable thing.

Design context still cannot be inferred from code. Code says what was built, not who it's for. Get audience / use cases / tone from the user or the project's spec/`.impeccable.md` before design work.

---

## Typography

**DO** drive every size from the Material type ramp; vary weight and role for hierarchy.
**DO** pair a distinctive **display** family with a refined **body** family — two `FontFamily` resources.
**DON'T** ship the platform defaults as the design: **Segoe UI** (WinUI) and **Roboto** (Material) are XAML's "Inter/Roboto/Arial." Replacing them deliberately is the single biggest de-genericizing move.
**DON'T** set literal `FontSize` / `FontWeight` off-ramp — it breaks the scale and dark/theme adaptation.
**DON'T** use monospace as shorthand for "technical/developer."
**DON'T** stack a large rounded `FontIcon`/`PathIcon` above every heading — the templated look.

**XAML gotcha (validated, in scaffolding rules):** variable-font TTFs render their default named instance on the Skia text stack; `FontWeight` does **not** select the `wght` axis. Ship one static TTF per weight and expose one `FontFamily` resource per file. Do not rely on a single variable font for a weight range.

---

## Color & theme

**DO** commit to a dominant surface + sharp accent; let Material's tonal roles carry it (`Surface`, `SurfaceVariant`, `Primary`, `Secondary`, `OnSurface`, `OnSurfaceVariant`).
**DO** express dark mode through `ThemeDictionaries`, not a second hand-built palette.
**DO** tint neutrals toward the brand hue by overriding the neutral tones in `ColorPaletteOverride.xaml` — subtle, cohesive.
**DON'T** put gray text on a colored surface — use that surface's `On*Variant` role instead (the Material equivalent of "a shade of the background color").
**DON'T** default to **dark mode + glowing accents** as a substitute for design decisions.
**DON'T** use the AI palette: cyan-on-dark, purple→blue gradients, neon on black. In WinUI this also means: don't lean on **Mica + Acrylic + the default accent purple** as the whole look.
**DON'T** reach for gradient text (`Opacity`-mask / Composition brush hacks) for "impact" — decorative, not meaningful.

**XAML gotcha (scaffolding rules):** custom DPs inside `DataTemplate`s don't receive `ThemeResource`, and custom theme-dictionary keys can fail even with `StaticResource`. Put app-specific semantic Colors/Brushes in a plain root-level dictionary merged in `App.xaml`; keep only Material role overrides in `ColorPaletteOverride.xaml`.

---

## Layout & space

XAML has no `clamp()` spacing, so rhythm is **deliberate**, not fluid. Define a spacing scale as `x:Double` resources (defer the scale itself to `userinterface-wiki-uno` → *Consistent Spacing Scale*) and vary it on purpose — tight groupings, generous separations — never one `Padding` value everywhere.

**DO** use asymmetry: `Grid` star columns, off-center emphasis, `HorizontalAlignment="Left"` for text.
**DON'T** wrap everything in a `Border`/`Card` — not every element needs a container.
**DON'T** nest `Card` in `Card` — flatten it.
**DON'T** build endless identical `ItemsRepeater` grids of same-sized icon+heading+text cards.
**DON'T** center everything — left-aligned + asymmetric reads as designed.

**XAML gotcha (validated LagoonSpecimen):** a `Grid` with non-`Stretch` `HorizontalAlignment` sizes to content, collapsing star columns that hold only empty/placeholder children to 0. Use `HorizontalAlignment="Stretch"` + `MaxWidth` so star columns fill regardless of empty cells.

---

## Visual details

**DON'T** apply glassmorphism/Acrylic/backdrop blur everywhere decoratively — reserve it for a purposeful moment (you have `LiquidGlassView` for the deliberate case).
**DON'T** ship rounded rectangles with a generic drop shadow — use the layered `ShadowContainer` stacks from `xaml-design-polish` (2–8% opacity, never pure black), and set `Shadows` inline per instance (a shared `ShadowCollection` via Style drops inner shadows — validated gotcha).
**DON'T** add a thick colored border on one side of a container — the web skill's "lazy accent" tell applies identically in XAML.
**DON'T** default to `ContentDialog` when an inline surface, `Flyout`, or `TeachingTip` fits — modals are lazy.

---

## Motion

Mechanics live in `xaml-design-polish` — use its numbers verbatim, do not invent competing ones:
- Easing: `EaseSmooth 0.22,1 0.36,1` (default), `EaseOut 0.17,1 0.32,1`, `EaseInOut 0.66,0 0.34,1`. The system default `ThemeTransition`/`VisualTransition` curve is the #1 tell — banned for anything expressive.
- Duration: fast 150 / normal 200 / slow 280 ms. Radius: 6 / 12 / 24.
- Press: scale to **0.98** (not 0.96/0.97 — those web values are superseded here).
- Premium entrance: `Opacity 0→1` + `Translation.Y 6→0` + Composition `GaussianBlur 2→0px`, ~280ms `EaseSmooth`.

Translated web rules:
**DO** put motion on state changes — entrance, exit, feedback — and concentrate it in one orchestrated load with staggered reveals over scattered micro-interactions.
**DON'T** animate `Width`/`Height`/`Margin`/`Padding` — use `Translation`/`Scale`/`Opacity` (Composition, off-thread) for 60fps. Height reveal uses the skill's recipe.
**DON'T** encode bounce in the curve — a `KeySpline` clamps to `[0,1]` and can't overshoot. A pop is a **value** overshoot (`scale 1→1.05→1`), or a Composition spring (`DampingRatio ~0.7`).
**DON'T** run per-item implicit animations on virtualized panels — use `ItemContainerTransitions`.
Gate all decorative motion on `UISettings().AnimationsEnabled`.

---

## Interaction

**DO** progressive disclosure — `Expander` for advanced options; reveal secondary actions in the `PointerOver` visual state.
**DO** design empty states that teach — in MVUX, the `FeedView` `NoneTemplate` should explain the interface, not say "nothing here."
**DO** optimistic UI — update `IState`/`IListState` immediately, sync after (MVUX makes this the natural path).
**DON'T** make every button `Filled`/primary — use the Material `Elevated`/`Outlined`/`Text` hierarchy.
**DON'T** repeat information the user can already see (redundant headers restating the page title).
**DON'T** strip focus visuals — keep `FocusVisualPrimaryBrush`/reveal focus; keyboard users need it.

---

## Responsive

**DO** adapt, don't shrink: `ResponsiveExtension` swaps property values per breakpoint; `ResponsiveView` swaps whole `DataTemplate`s; `AdaptiveTrigger` + `VisualStateManager` restructures the shell. Use `SafeArea` for insets.
**DON'T** amputate functionality on small form factors — restructure the layout, keep the capability.

**XAML gotcha (scaffolding rules):** `DrawerControl` is not a docked always-open panel — dock panels as `Grid` columns on wide layouts and use `DrawerControl` only for transient/narrow cases.

---

## UX writing

Unchanged: make every word earn its place; never restate what the UI already shows. (Content hard rules — "Uno Platform" never "Uno", no em dashes, LLM-agnostic — apply only to outward-facing text, not in-app microcopy, but keep them for anything published.)

---

## The AI-slop test (XAML edition)

If someone saw this and instantly said "an AI/template made this," that's the failure. The XAML fingerprints of generated 2024–2025 work:

- Untouched **Segoe UI / Roboto** as the type identity.
- **Mica + Acrylic + default accent purple** as the entire visual language.
- A stock **`NavigationView`** left rail with no adaptation.
- Everything in identical **`Card`** grids; system-default transitions; pure-black drop shadows.
- Dark mode with glowing accents standing in for design decisions.

A distinctive XAML interface makes someone ask "how was this made?" — not "which AI made this?"

---

## Implementation principle

Match code complexity to the vision: maximalism earns elaborate Composition/Storyboard work; minimalism demands restraint and precise spacing/type. Vary direction across projects — light/dark, different families, different aesthetics — never converge on one default look.
