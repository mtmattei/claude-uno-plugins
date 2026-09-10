# Runtime SkSL Shaders, Image Filters, and Backdrop Effects

Verified against the `4.148.0` managed binding. These APIs exist in both 3.119.x and 4.148.0; v4 brings the m148 engine and the GPU/CPU performance gains under them.

## Runtime shaders (SkSL)

The runtime-effect family compiles SkSL at runtime and feeds uniforms/children:

- `SKRuntimeEffect` - compiled effect.
- `SKRuntimeShaderBuilder` - builds an `SKShader` from a shader effect.
- `SKRuntimeColorFilterBuilder` - builds an `SKColorFilter`.
- `SKRuntimeBlenderBuilder` - builds an `SKBlender`.

```csharp
// Build ONCE and cache. See the disposal gotcha below.
const string Sksl = """
uniform float2 iResolution;
uniform float  iTime;
half4 main(float2 fragCoord) {
    float2 uv = fragCoord / iResolution;
    return half4(uv.x, uv.y, 0.5 + 0.5 * sin(iTime), 1.0);
}
""";

var effect = SKRuntimeEffect.CreateShader(Sksl, out var errors);
if (effect is null) throw new InvalidOperationException(errors);
var builder = new SKRuntimeShaderBuilder(effect); // cache this field
```

Per frame (uniform updates are cheap; building the effect is not):

```csharp
protected override void RenderOverride(SKCanvas canvas, Size area)
{
    _builder.Uniforms["iResolution"] = new[] { (float)area.Width, (float)area.Height };
    _builder.Uniforms["iTime"] = _time; // snapshot on UI thread, read here
    using var shader = _builder.Build();
    using var paint = new SKPaint { Shader = shader };
    canvas.DrawRect(0, 0, (float)area.Width, (float)area.Height, paint);
}
```

`_paint` can also be cached and have its `Shader` reassigned; only the per-frame `Build()` result is short-lived.

## Image filters that exist in 4.148.0

`SKImageFilter` static factories confirmed in the binding:

- `CreateBlur(sigmaX, sigmaY, tileMode, input?, cropRect?)`
- `CreateShader(SKShader?, dither?, cropRect?)` - wraps a shader (including a runtime-shader-built `SKShader`) as a filter.
- `CreateDisplacementMapEffect(xChannel, yChannel, scale, displacement, input?, cropRect?)`
- `CreateArithmetic(k1, k2, k3, k4, enforcePMColor, background, foreground?, cropRect?)`
- `CreateBlendMode(SKBlendMode | SKBlender, background, foreground?, cropRect?)`
- `CreateMatrixConvolution(...)`

There is **no** `RuntimeShader` image filter (no `SkImageFilters::RuntimeShader` binding). Route SkSL into the filter graph through `SKImageFilter.CreateShader(builder.Build())` and use it as the displacement input of `CreateDisplacementMapEffect` when you need refraction.

## Backdrop effects (Liquid Glass style) with SKCanvasElement

`SKCanvasElement.RenderOverride` draws on the same canvas the window renders to, so `SaveLayer` with a backdrop filter genuinely samples the live app content behind the element. This is what makes glass/refraction effects possible without re-rendering the scene yourself.

```csharp
protected override void RenderOverride(SKCanvas canvas, Size area)
{
    var rect = SKRect.Create((float)area.Width, (float)area.Height);

    // _backdrop is a cached SKImageFilter graph (built when DP inputs change, not per frame).
    var rec = new SKCanvasSaveLayerRec
    {
        Bounds = rect,
        Backdrop = _backdrop, // samples live content behind the element
    };
    canvas.SaveLayer(in rec);
    canvas.Restore();

    // draw rim/highlight on top
    canvas.DrawRoundRect(new SKRoundRect(rect, 24), _rimPaint);
}
```

Refraction recipe (displacement driven by an SkSL shader):

1. Build a normal/height SkSL shader with `SKRuntimeShaderBuilder`.
2. Wrap it: `var disp = SKImageFilter.CreateShader(builder.Build());`
3. Use it as displacement: `SKImageFilter.CreateDisplacementMapEffect(R, G, scale, disp)`.

Chromatic aberration: three displacement filters at split scales, each through a channel-isolating color matrix (`SKColorFilter.CreateColorMatrix`), recombined with `SKImageFilter.CreateArithmetic`.

## Mesh / vertex rendering

The mesh primitive in SkiaSharp 4.148.0 is `SKVertices`, drawn with `canvas.DrawVertices`:

```csharp
using var vertices = SKVertices.CreateCopy(
    SKVertexMode.Triangles, positions, texs, colors, indices);
canvas.DrawVertices(vertices, SKBlendMode.Modulate, paint);
```

There is no `SKMesh` (custom-mesh-with-SkSL) type in the GA managed binding. For 3D, you project/light/depth-sort in your own CPU or shader code and emit triangles through `SKVertices`, or drive a full SkSL shader. See `references/04-gotchas-performance.md`.

## v4 typography APIs

Variable fonts (OpenType axes):

```csharp
var coords = new[]
{
    new SKFontVariationPositionCoordinate { Axis = new SKFourByteTag('w','g','h','t'), Value = 650 },
    new SKFontVariationPositionCoordinate { Axis = new SKFourByteTag('w','d','t','h'), Value = 100 },
};
var args = new SKFontArguments { VariationDesignPosition = coords };
using var stream = new SKManagedStream(fontFileStream);
using var typeface = SKTypeface.FromStream(stream, args);
```

Color font palettes (emoji / icon fonts):

```csharp
// pick a built-in palette index
using var alt = baseTypeface.Clone(paletteIndex: 1);

// or override specific palette entries
var overrides = new[]
{
    new SKFontPaletteOverride { Index = 0, Color = SKColors.Crimson },
};
var args = new SKFontArguments { PaletteIndex = 0, PaletteOverrides = overrides };
```

Animated WebP encoding:

```csharp
var frames = pixmaps
    .Select(p => new SKWebpEncoderFrame { Pixmap = p, Duration = TimeSpan.FromMilliseconds(40) })
    .ToArray();
var data = SKWebpEncoder.EncodeAnimated(frames, new SKWebpEncoderOptions());
```
