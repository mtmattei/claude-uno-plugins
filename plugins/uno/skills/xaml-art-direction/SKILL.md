---
name: xaml-art-direction
description: Aesthetic direction for Uno Platform / WinUI XAML apps — commit to a distinctive visual identity (type, palette tokens, signature element) before writing markup, so the result doesn't read as a templated Material/Fluent default. Use when building or restyling any XAML surface where the look matters: new apps, showcase pages, demos, samples headed for an audience, or when the user says a design feels generic, templated, or "AI-made". Owns direction only — motion numbers live in xaml-design-polish, per-detail correctness in userinterface-wiki-uno, Material mechanics in uno-material. Do NOT use to elevate an already-approved design without changing its identity — that is gold-standard-pass.
---

# XAML Art Direction

Approach this as the design lead at a small studio known for giving every client a visual identity that could not be mistaken for anyone else's. The client has already rejected templated proposals. Make deliberate, opinionated choices about palette, typography, and layout that are specific to this brief, and take one real aesthetic risk you can justify.

This is the *direction* layer for XAML surfaces. Role split — do not duplicate:
- **How motion/interaction feels** (curves, durations, press, entrances) → `xaml-design-polish`; its numbers are authoritative.
- **Whether each XAML detail is correct** (states, focus, virtualization, spacing scale, reduced motion) → `userinterface-wiki-uno`.
- **Material color roles, type ramp, control styles** → `uno-material`.
- **Elevating an existing, approved design without changing its identity** → `gold-standard-pass`. If the brief is "make this finished", not "make this distinctive", that skill owns it and this one stands down.
- **This skill** — what the design *is*: subject grounding, tokens, typography identity, the signature element, and the anti-slop calibration. On conflict about a mechanism, the owning skill above wins.

## Ground it in the subject

If the brief does not pin down what the product or subject is, pin it yourself before designing: name one concrete subject, its audience, and the surface's single job, and state your choice. The subject's own world — its materials, instruments, artifacts, and vernacular — is where distinctive choices come from. A lawn-care app's identity came from mowing stripes and roadside signage type; a synth app's would come from panel silkscreens and jacks. Build with the brief's real content throughout; realistic sample data is part of the design.

## Process: plan, calibrate, then build

Work in two passes. **Pass 1 — a compact design plan before any XAML:**

- **Color:** 4–6 named hex values (canvas, card, ink, one committed primary, 1–2 supports, semantic alert). Semantic colors are semantic-only — an amber that means "delay" never decorates.
- **Type:** 2–3 roles — a characterful display face used with restraint, a complementary body face, optionally a utility face (mono for data). Name the faces and weights.
- **Composition:** a named layout archetype derived from the subject's own artifacts — a ledger page, a chart, a bin, a rail, a bench, a ticket board — plus an ASCII wireframe of it at both wide and narrow. The archetype is a commitment like the palette is: name it and say which artifact it comes from. Card/row anatomy is part of the commitment (where the key figure sits, where status lives, where actions go) — derive it from the archetype, not from habit. Asymmetry and left-alignment read as designed; centered-everything reads as template.
- **Signature:** the single element this surface will be remembered by, drawn from the subject's vernacular. One per app.

**Pass 2 — calibrate before building.** Ask: would I have produced this plan for any similar prompt? Generated design clusters around known defaults — on the web: cream + serif + terracotta, near-black + acid accent, broadsheet hairlines; in XAML: untouched Segoe/Roboto, Mica/Acrylic + default accent, a stock NavigationView rail, identical card grids, dark mode + glow. **Composition has its own default cluster** (validated 3× convergence, DesignSkillEval solo runs, 2026-08): identity band across the top with one filled primary top-right, card list on the left two-thirds, stacked panel rail on the right third, three-segment sticky nav on narrow, and inside every card a big figure left + status chip top-right + text actions along the bottom. If the wireframe matches that formula, the archetype wasn't derived — go back to the subject's artifacts. The same goes for accessory defaults this skill itself tends toward: mono-for-data in every plan and a red-orange in the hot slot — each must re-earn its place from this subject or be replaced. Where the brief pins an axis (e.g. "warm off-white and deep green"), follow it exactly — the brief always wins. Where an axis is free, spend that freedom away from the defaults. Revise the generic parts of the plan, say what changed, then build from the revised plan and derive every color and type decision from it.

**Spend boldness in one place.** The signature element is the one memorable thing; keep everything around it quiet — hairline borders, restrained shadows, one filled-primary action per view. Before shipping, look in the mirror and remove one accessory.

## XAML execution — mechanisms and proven recipes

Web concepts translate; a literal port is wrong. Use the platform mechanism:

| Intent | XAML mechanism |
|---|---|
| Design tokens | `x:Double`/`Color`/`Brush` resources in a plain root-level dictionary merged in App.xaml (custom keys in ThemeDictionaries resolve unreliably; keep Material role overrides separate in `ColorPaletteOverride.xaml`) |
| Custom fonts | Static per-weight TTFs in `Assets/Fonts`, one `FontFamily` resource per file. Variable TTFs render only their default instance on the Skia text stack — never rely on `FontWeight` to select a `wght` axis. Verify internal family names from the TTF name tables before wiring. |
| Brand carried by stock controls | Override Material roles (`Primary`, `Surface`, neutrals tinted toward the brand hue) in `ColorPaletteOverride.xaml` so CheckBox/Slider/ComboBox inherit the palette instead of stock purple |
| Data-driven brushes in templates | Mirror the token palette as a static C# class; custom DPs inside DataTemplates don't receive `ThemeResource` |
| Signature patterns (stripes, grids, fields) | Hard-stop `LinearGradientBrush` or a small `Path` — one brush beats twelve rectangles |
| Icons that might tofu on Skia | `Path`/`Ellipse` geometry over font glyphs |
| Fluid type / `clamp()` | No equivalent — discrete breakpoints via `ResponsiveExtension` / `AdaptiveTrigger` + VSM |
| `prefers-reduced-motion`, easing, entrances | Defer to `xaml-design-polish` (gate on `UISettings.AnimationsEnabled`) |

Quality floor, built without announcing it: responsive down to phone width, statuses never color-alone, large touch targets, visible keyboard focus, high contrast on the chosen canvas.

## Writing is design material

Copy gets the same intentionality as spacing. Plain verbs from the user's side of the screen ("Start route", not "Initiate"); an action keeps its name through the whole flow; errors say what happened and what to do; an empty state is an invitation to act ("No jobs booked — capacity for 7 lawns"). Explain computed results — a price shows its formula, never unexplained automation.

## Example (validated: DesignSkillEval 07, 2026-07)

Brief: lawn-crew daily ops app, "warm off-white, deep greens" pinned. Plan: Hay `#F5F3EA` canvas, Pine `#2F6B3C` single primary, Moss ink, semantic-only amber; Barlow Condensed display (truck-signage vernacular) + Barlow body + Plex Mono for times/prices/mileage; signature = mowed-stripe language (header progress segments that "mow" as jobs complete, completed cards flip to a hard-stop stripe brush, calendar stripe pips). Boldness spent on the stripe system; everything else hairlines and one pine button per view. Scored highest visual identity of a 7-build comparison.

## Anti-pattern

Repainting stock Material and calling it direction: keep Roboto, swap `Primary` to the brief's color, add a stats-row header and identical rounded cards. It compiles, it's "on brand", and it's indistinguishable from every other generated app — the 4.8/10 control build in the same comparison did exactly this. Direction means the type, tokens, and signature were *chosen for this subject*, and you can say why.

## Full DO/DON'T reference

The complete per-area DO/DON'T tables (typography, color, layout, visual details, motion, interaction, responsive) migrated from the former `~/.claude/rules/xaml-design-principles.md` live in `references/design-principles-full.md`. Read it when doing a full design pass; the sections above are the operating summary.

## When NOT to use

Internal fixtures, throwaway repro projects, or when the user explicitly wants stock Material/Fluent. Do not re-derive motion values or spacing scales here — invoke the owning skills alongside this one.
