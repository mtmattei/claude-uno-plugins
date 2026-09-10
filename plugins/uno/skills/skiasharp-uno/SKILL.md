---
name: skiasharp-uno
description: "SkiaSharp 2D/GPU drawing inside Uno Platform apps, with emphasis on SkiaSharp 4 (GA 4.148.0, June 2026). Covers SKCanvasElement vs SKXamlCanvas, opting into SkiaSharp 4 in an Uno app, version/native-asset pinning, runtime SkSL shaders, image filters and backdrop effects, SKVertices meshes, variable fonts and color palettes, and the runtime gotchas. Use when: (1) Adding custom Skia drawing to an Uno Platform app, (2) Choosing SKCanvasElement vs SKXamlCanvas, (3) Upgrading or pinning SkiaSharp (especially to v4) in an Uno project, (4) Writing SkSL runtime shaders / image-filter or backdrop effects on Skia targets, (5) Diagnosing native/managed version mismatches or per-frame shader crashes, (6) Using v4 features (variable fonts, color palettes, animated WebP). Do NOT use for: general Uno project setup (see uno-platform-agent), the Skia renderer pipeline at large (see uno docs how-uno-works), or Material theming (see uno-material)."
license: "Apache 2.0 (patterns derived from Uno Platform and SkiaSharp documentation and source)"
metadata:
  version: "1.0.0"
  skiasharp_ga: "4.148.0"
  verified_against: "mono/SkiaSharp tag v4.148.0 managed binding source, 2026-06-30"
---

# SkiaSharp with Uno Platform

Patterns for custom Skia drawing inside Uno Platform apps, current to **SkiaSharp 4.0 GA (`4.148.0`, June 2026)**. Uno Platform is now a **co-maintainer of SkiaSharp** alongside Microsoft's .NET team, so cross-platform rendering needs are represented directly in the release process.

Facts here were verified against the `v4.148.0` managed binding source, not release-note summaries.

## Critical Rules

**Pick the right canvas element**
- Prefer **`SKCanvasElement`** (`Uno.WinUI.Graphics2DSK`) for new Uno work on Skia targets. It draws onto the window's existing internal Skia canvas, is hardware-accelerated when the app renders with OpenGL, and skips the extra buffer copy.
- Use **`SKXamlCanvas`** (`SkiaSharp.Views.Uno.WinUI`) only when you need to run on a non-Skia target (for example WinAppSDK without Skia rendering) or you are porting existing `OnPaintSurface` code from Xamarin.Forms / WinUI. It is not hardware-accelerated on Skia targets and copies buffers.
- `SKCanvasElement` is only supported where Skia rendering is active. Always guard with `SKCanvasElement.IsSupportedOnCurrentPlatform()`; constructing it where unsupported throws.

**SkiaSharp 4 is opt-in on Uno today**
- Uno.Sdk still bundles SkiaSharp **3.119.x** by default. v4 is experimental opt-in, not the default.
- Opt in with the MSBuild property `<SkiaSharpVersion>4.148.0</SkiaSharpVersion>`, or pin every SkiaSharp/HarfBuzzSharp package (and its native-asset packages) explicitly to the same version.
- A managed/native version mismatch is the first thing to check on a runtime crash. The bundled native lib must match the managed package version. See `references/02-skiasharp-4-adoption.md`.

**Never build Skia filter/shader graphs per frame on the render thread**
- `RenderOverride` runs on the render thread. Building `SKImageFilter` DAGs or `SKRuntimeEffect` objects inside it every frame is the most common cause of jank and native crashes.
- Create and **cache** `SKRuntimeShaderBuilder` once. Disposing it per frame frees native state shared with its `SKRuntimeEffect` and the next render access-violates (`0xC0000005`). Snapshot DependencyProperty values to plain fields on the UI thread; render from those.

**Reuse paints, dispose Skia objects**
- Reuse `SKPaint`, `SKShader`, and filter instances across frames; do not allocate them in the render loop.
- Dispose Skia objects (`SKPaint`, `SKSurface`, `SKImage`, builders) when their owner is torn down.

## SkiaSharp 4: what is actually in GA (verified)

Present in the `4.148.0` managed binding:
- **Skia engine m148** under the hood (rendering, codec, performance, security upstream work).
- **Variable fonts**: `SKFontArguments.VariationDesignPosition` (`SKFontVariationPositionCoordinate`), `CollectionIndex`.
- **Color font palettes**: `SKFontArguments.PaletteIndex` / `PaletteOverrides` (`SKFontPaletteOverride`), `SKTypeface.Clone(int paletteIndex)`.
- **Animated WebP**: `SKWebpEncoder.EncodeAnimated(ReadOnlySpan<SKWebpEncoderFrame>, ...)`.
- **Runtime SkSL shaders**: `SKRuntimeEffect`, `SKRuntimeShaderBuilder`, `SKRuntimeColorFilterBuilder`, `SKRuntimeBlenderBuilder`.
- **Image filters**: `CreateShader`, `CreateDisplacementMapEffect`, `CreateArithmetic`, `CreateBlendMode`, `CreateBlur`, `CreateMatrixConvolution`.
- **Mesh primitive**: `SKVertices.CreateCopy(SKVertexMode, positions, texs, colors, indices)` drawn via `canvas.DrawVertices(...)`.
- **GPU backends**: OpenGL, Direct3D, Metal, Vulkan (`GRContext` and the `GR*BackendContext` types).

Breaking / behavior changes to plan for:
- **Legacy obsolete APIs are now hard compile errors.** Old `SKPaint` text-measuring/state APIs that warned in 3.x will not build under v4. Migrate to `SKFont` for text.
- **Native singleton lifecycle rework** fixes a class of use-after-free crashes during GC finalization of managed wrappers mid native call.
- `SKPixmap` stride and coordinate fixes.

Not in GA (do not promise these for `4.148.0`):
- **Graphite GPU backend**: preview only (`4.150.0-preview.2.1`).
- **`SKMesh`** (a custom-mesh-with-SkSL primitive): not present in the `4.148.0` managed binding. The mesh primitive that exists is `SKVertices` + `DrawVertices`. See `references/04-gotchas-performance.md`.

## Decision: SKCanvasElement vs SKXamlCanvas

| Need | Use |
|------|-----|
| New custom drawing on Skia desktop / Skia-rendered mobile | `SKCanvasElement` |
| Hardware acceleration, no buffer copy | `SKCanvasElement` |
| Backdrop effects that sample live app content behind the element | `SKCanvasElement` (`RenderOverride` draws on the window canvas) |
| Must run on WinAppSDK without Skia rendering | `SKXamlCanvas` |
| Porting existing `OnPaintSurface` code | `SKXamlCanvas` (least change) |

## Related Skills

| Skill | Use instead when... |
|-------|--------------------|
| `uno-platform-agent` | General project setup, Uno.Sdk, UnoFeatures, custom-control architecture |
| `uno-material` | Theming, colors, typography styles (not raw Skia drawing) |
| `xaml-design-polish` | Motion feel and timing numbers for the interaction wrapping the canvas |
| `winui-xaml` | XAML layout, binding, rendering of non-Skia visuals |
| `dotnet-csharp` | General C# performance, memory, async correctness |

## Detailed References

Read the reference matching the task:

- [references/01-integration-skcanvaselement.md](references/01-integration-skcanvaselement.md) - Read when adding `SKCanvasElement` (or `SKXamlCanvas`) to a page: setup, `RenderOverride`, renderer separation, invalidation, animation loop, platform guards, WinAppSDK config.
- [references/02-skiasharp-4-adoption.md](references/02-skiasharp-4-adoption.md) - Read when opting into or pinning SkiaSharp (especially v4) in an Uno project: `SkiaSharpVersion`, native-asset packages, Uno.Sdk matching, Lottie/Skia version assert, migrating off obsolete-to-error APIs.
- [references/03-runtime-shaders-effects.md](references/03-runtime-shaders-effects.md) - Read when writing SkSL runtime shaders, image-filter effects, or backdrop/refraction effects (Liquid Glass style) with `SKRuntimeShaderBuilder` and `SaveLayer`.
- [references/04-gotchas-performance.md](references/04-gotchas-performance.md) - Read when diagnosing per-frame crashes, version mismatches, render-thread jank, or deciding between `SKVertices` and the absent `SKMesh`.
