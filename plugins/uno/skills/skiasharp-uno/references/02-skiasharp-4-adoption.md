# Adopting and Pinning SkiaSharp 4 in an Uno Platform App

SkiaSharp 4.0 GA is `4.148.0` (tagged June 23, announced June 29 2026). Uno.Sdk still bundles SkiaSharp **3.119.x** by default; v4 is experimental opt-in. Uno Platform is a co-maintainer of SkiaSharp, so v4 support lands in Uno on a predictable cadence.

## Opt in: the simple path

Set the SkiaSharp version with the MSBuild property in the app `.csproj`. Uno.Sdk flows that version to the SkiaSharp packages it manages:

```xml
<PropertyGroup>
  <SkiaSharpVersion>4.148.0</SkiaSharpVersion>
</PropertyGroup>
```

After changing it: stop any running instance, `dotnet restore`, then `dotnet build` and run. A managed/native mismatch surfaces at runtime, not build time, so launch the app and read the console before assuming success.

## Opt in: explicit package pinning

When you need fine control (or the `SkiaSharpVersion` property is not honored by your Uno.Sdk version), pin every SkiaSharp and HarfBuzzSharp package, including the native-asset packages for each target you build, to the same version. Mismatched native assets are the usual cause of `DllNotFoundException` / startup access violations.

```xml
<ItemGroup>
  <PackageReference Update="SkiaSharp" Version="4.148.0" />
  <PackageReference Update="SkiaSharp.NativeAssets.Win32" Version="4.148.0" />
  <PackageReference Update="SkiaSharp.NativeAssets.Linux" Version="4.148.0" />
  <PackageReference Update="SkiaSharp.NativeAssets.macOS" Version="4.148.0" />
  <PackageReference Update="SkiaSharp.NativeAssets.WebAssembly" Version="4.148.0" />
  <PackageReference Update="HarfBuzzSharp" Version="8.3.1.1" />
  <PackageReference Update="HarfBuzzSharp.NativeAssets.Win32" Version="8.3.1.1" />
</ItemGroup>
```

Use `Update=` (not `Include=`) so you retarget the references Uno.Sdk already brings in rather than duplicating them. Only list the native-asset packages for targets you actually build.

Verify the resolved version:

```powershell
dotnet build -bl
# or inspect the lock / asset graph:
dotnet list package --include-transitive | Select-String "SkiaSharp"
```

## Uno.Sdk matching for preview SkiaSharp

When pinning a *preview* SkiaSharp (for example tracking `4.150.0-preview.x` for the Graphite backend) ahead of what Uno stable bundles, the managed package must match the native lib the running Uno.Sdk ships. The reliable combination is:

- A matching **Uno.Sdk dev build** whose bundled native SkiaSharp matches the managed package version.
- `<UnoDisableLottieSkiaVersionCheck>true</UnoDisableLottieSkiaVersionCheck>` to suppress Uno's Lottie/Skia version assert when versions are intentionally ahead.
- Cross-reference the official SkiaSharp Uno gallery sample (`samples/Gallery/Uno/SkiaSharpSample.Uno.csproj` at the SkiaSharp tag) for the Uno.Sdk version that pairs with that SkiaSharp build.

For GA `4.148.0` on a current Uno.Sdk, the dev-build dance is usually unnecessary. Reach for it only when a native/managed mismatch appears at runtime.

## Migrating off obsolete-to-error APIs (v4 breaking change)

v4 turns previously deprecated members into **hard compile errors**. The most common breakage is text measurement and paint state that moved from `SKPaint` to `SKFont`.

```csharp
// v3 (deprecated, now a build error in v4):
// var width = paint.MeasureText("hi");
// paint.TextSize = 24;

// v4:
using var font = new SKFont { Size = 24 };
var width = font.MeasureText("hi", out var bounds);
canvas.DrawText("hi", x, y, SKTextAlign.Left, font, paint);
```

Also affected: glyph/run helpers and assorted state readers on `SKPaint`. Resolve them by moving to the `SKFont` / `SKTextBlob` equivalents. Build first; the compiler enumerates every site for you.

## Lifecycle change to be aware of

v4 reworks native singleton lifetimes with proper reference counting, fixing use-after-free crashes that could happen when the GC finalized a managed wrapper during an in-flight native call. No code change is required, but if you previously worked around such crashes with manual `GC.KeepAlive` hacks around singletons, you can usually remove them.

## What you get for adopting v4

- Skia engine m148: upstream rendering, codec, performance, and security work with no code changes.
- GPU-side wins on the work that dominates modern UI (elevated cards, drop shadows, layered surfaces) and large CPU gains on procedural shaders.
- Variable fonts, color font palettes, animated WebP, and zero-copy stream conversion (see `references/03-runtime-shaders-effects.md` for the typography APIs).

## Verification checklist

- App launches on every target you build (desktop first, then mobile/WASM).
- Console shows no `DllNotFoundException` or SkiaSharp version assert.
- A trivial `SKCanvasElement` draw renders.
- `dotnet list package` shows a single resolved SkiaSharp version across managed and native-asset packages.
