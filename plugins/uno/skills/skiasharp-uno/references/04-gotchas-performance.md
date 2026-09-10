# SkiaSharp + Uno: Gotchas and Performance

Hard-won runtime behavior. Most of these cost hours the first time.

## Cache `SKRuntimeShaderBuilder`; never per-frame `using`

`SKRuntimeShaderBuilder` must be created once and kept as a field. Disposing it (for example a per-frame `using`) frees native state shared with its `SKRuntimeEffect`. The next render access-violates with `0xC0000005` inside `sk_runtimeeffect_get_uniform_byte_size`.

- Symptom: first frame renders fine, crash on the next invalidation (often the first drag/resize).
- Fix: build the effect and builder once; only `Build()` the `SKShader` per frame (that result is short-lived and disposable).

## Never build Skia filter graphs on the render thread

`RenderOverride` runs on the render thread. Building `SKImageFilter` DAGs there every frame causes jank and, with runtime-shader inputs, native crashes.

- Cache the filter graph against the values it was built with. Rebuild only when those inputs change.
- Snapshot DependencyProperty values to plain fields on the UI thread, then read the fields inside `RenderOverride`. Do not touch DPs from the render thread.

## Reuse paints; dispose on teardown

- Allocate `SKPaint`, `SKShader`, gradients, and filters once; reuse across frames. Per-frame allocation churns the GC and the native heap.
- Dispose Skia objects when the owning control/renderer is `Unloaded`/disposed. Wire `Unloaded` to stop timers and dispose the renderer.

## Native/managed version mismatch is the first thing to check

A SkiaSharp crash at startup or first draw is usually a version mismatch, not your code.

- The bundled native lib must match the managed package version. When pinning v4 or a preview, pin every native-asset package to the same version (see `references/02-skiasharp-4-adoption.md`).
- For preview SkiaSharp ahead of Uno stable, verify the Uno.Sdk dev build matches before patching managed code, and set `<UnoDisableLottieSkiaVersionCheck>true</UnoDisableLottieSkiaVersionCheck>`.

## `SKMesh` is not in the GA managed binding

The `4.148.0` managed binding has no `SKMesh` (custom-mesh-with-SkSL) type. The mesh primitive that exists is `SKVertices` (`CreateCopy(SKVertexMode, ...)` drawn with `canvas.DrawVertices`).

- For triangle meshes, gradient meshes, or simple textured 3D, use `SKVertices`.
- For procedural/3D effects, drive a full SkSL shader via `SKRuntimeShaderBuilder` and do projection/lighting/depth-sort in shader or CPU code.
- If a community "SkMesh"-style 3D sample is the reference, confirm which SkiaSharp build it used. It is likely `SKVertices` + `DrawVertices`, or a custom P/Invoke, not a GA `SKMesh` binding. Do not promise `SKMesh` for `4.148.0`.

## Graphite is preview-only

The Graphite GPU backend ships in `4.150.0-preview.2.1`, not in GA `4.148.0`. Do not assume Graphite when targeting the stable release. The GA GPU backends are OpenGL, Direct3D, Metal, and Vulkan.

## v4 obsolete-to-error will break a 3.x build

Upgrading a 3.x project to v4 can fail to compile because deprecated `SKPaint` text/state APIs are now hard errors. This is expected. Build, let the compiler list every site, and migrate to `SKFont` / `SKTextBlob`. See `references/02-skiasharp-4-adoption.md`.

## Platform support guard

`SKCanvasElement` only works where Skia rendering is active (`netX.0-desktop`, or other targets with the `SkiaRenderer` feature). Always branch on `SKCanvasElement.IsSupportedOnCurrentPlatform()` and provide a non-Skia fallback rather than letting the constructor throw.

## WinAppSDK build configuration

On WinAppSDK, `SKCanvasElement` needs an `x64` or `ARM64` solution configuration. `AnyCPU` will not run it. Create the configuration in Configuration Manager and assign it to the Uno project.

## Animation invalidation

`SKCanvasElement` caches its draw and only redraws on `Invalidate()`. For continuous animation, drive `Invalidate()` from a UI-thread `DispatcherTimer` (~16 ms) or a frame callback, and advance simulation state outside `RenderOverride`. Stop the timer on `Unloaded`.
