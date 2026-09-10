---
name: uno-scaffolding
description: Uno Platform / .NET scaffolding and XAML rules reference — SDK versions, `dotnet new unoapp` setup, MVUX vs MVVM choice, XAML binding style ({Binding} for MVUX, x:Bind otherwise), region-based navigation, styling rules, platform strategy, the Extensions stack, SkiaSharp 4 capability notes and preview pinning combos, the skill-to-task map, WinUI plugin selection, and a long list of validated runtime gotchas. Use when scaffolding a new Uno app, choosing MVUX vs MVVM, deciding binding style, pinning a preview SkiaSharp, picking which Uno skill applies to a task, or diagnosing a runtime behaviour that smells like a known Uno/Skia quirk.
license: MIT
---

# Uno Scaffolding & XAML Rules

This skill carries the full scaffolding and XAML rules reference for Uno Platform work.

**Read `references/uno-scaffolding-rules.md` now.** It is the authoritative content; everything below is only a map of what is in it.

## What the reference covers

| Section | Use it when |
|---|---|
| SDK Versions, New App Scaffolding | Creating a new Uno app, choosing a TFM, running `uno-check` |
| MVUX vs MVVM | Deciding app architecture before writing any model |
| XAML Binding | Choosing `{Binding}` vs `x:Bind` (MVUX forces `{Binding}`) |
| Navigation (Region-Based) | Wiring `Navigation.Request` / `Region.Attached`, registering routes |
| Styling Rules | Colors, type scale, lightweight styling keys |
| Platform Strategy, Extensions Stack | Targeting, DI/hosting/auth/HTTP/config/logging choices |
| SkiaSharp 4 Capability Note | Any claim about SkiaSharp capabilities (note: no `SKMesh` in 4.148.0 GA; use `SKVertices`) |
| Preview SkiaSharp + Uno | Pinning a preview SkiaSharp against a matching Uno.Sdk dev version |
| Skill-to-Task Mapping | Choosing which Uno skill to invoke for a task |
| Runtime Gotchas | A long validated list — check here before debugging anything that looks like a framework bug |
| WinUI Plugin Selection | Which `winui:*` skills are safe in Uno sessions |

## Precedence

The reference wins over draft implementation. When a draft contradicts it, rewrite the draft.
