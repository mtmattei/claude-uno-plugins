---
name: uno-audit
description: Full codebase pass for an Uno Platform project - best practices, dead code, comment cleanup, performance, stack currency - with runtime verification. Use whenever asked to audit, review, clean up, optimize, or apply best practices to an Uno Platform codebase, or before pushing a demo/sample repo. Do NOT use for design-quality work on an existing surface (spacing, hierarchy, states, fit and finish) — that is gold-standard-pass.
---

# Uno Audit

The standard audit pass, so it never has to be dictated again. Guidance from invoked skills is binding: apply it, cite it while writing (per CLAUDE.md), and rewrite drafts that contradict it.

## Steps

1. **Invoke the relevant skills first** - `uno-platform-agent` always; `mvux`, `uno-navigation`, `uno-material`, `uno-toolkit`, `winui-xaml`, `dotnet-csharp` as the codebase touches each area. State per skill which pattern is being applied.
2. **Stack currency:** confirm .NET 10 TFMs and latest stable Uno.Sdk (check NuGet, never assume from habit). Flag preview pins that have a stable now.
3. **Code hygiene:** remove dead code, unused usings, duplicate/redundant code, narration comments (comments that describe the next line or justify a change). Keep constraint comments.
4. **Pattern conformance:**
   - MVUX surfaces use `{Binding}`, never `x:Bind` (see the `uno-scaffolding` skill).
   - Toolkit/Extensions components before custom controls or helpers - most "small helper" cases already have one.
   - No hardcoded hex colors or font sizes; Material resources and type-scale styles only.
   - Region-based navigation via XAML attached properties, no code-behind navigation.
   - Check against the Runtime Gotchas list in `uno-scaffolding.md`.
5. **Performance pass:** virtualization on lists, per-frame allocations, Skia object caching, startup work off the UI path.
6. **Build gate:** zero warnings. Then **runtime verify** the touched surfaces via `uno-verify` - an audit that only compiles is half done.
7. **Ship (only when asked):** conventional commit(s), push to the named repo, refresh README. Never commit or push unprompted.

## Output

Per CLAUDE.md implementation format: What I did / Files touched / Build+runtime result / Unresolved questions. Group findings by category with file:line references.

## Example

> "uno-audit then push to github.com/mtmattei/Build-Samples"
→ skills invoked and cited, 14 cleanups across 6 files, Uno.Sdk bumped, build clean, 3 screens re-verified via uno-app, conventional commits, pushed, README updated.

## Anti-pattern

Invoking the skills, then writing code from prior assumptions anyway. Invocation without application is worse than skipping the skill - it creates false confidence that guidance is loaded.
