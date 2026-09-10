# SkiaSharp Integration in Uno Platform (SKCanvasElement / SKXamlCanvas)

## Which element

`SKCanvasElement` (package `Uno.WinUI.Graphics2DSK`) is the modern choice. Uno.Sdk references the package automatically when the `Skia` or `SkiaRenderer` UnoFeatures are present, and always on the `netX.0-desktop` target. It draws onto the window's existing internal Skia canvas (hardware-accelerated when the app renders with OpenGL) and skips the buffer copy that `SKXamlCanvas` performs.

`SKXamlCanvas` (package `SkiaSharp.Views.Uno.WinUI`) still exists and is the right pick when you must run on a non-Skia target (for example WinAppSDK without Skia rendering) or you are porting existing `OnPaintSurface` code. It is not hardware-accelerated on Skia targets.

## SKCanvasElement: control + renderer split

Keep drawing logic out of the `FrameworkElement`. The element owns lifecycle and invalidation; a plain renderer owns the pixels. This keeps the renderer unit-testable and lets you swap rendering strategies.

```csharp
using SkiaSharp;
using Uno.WinUI.Graphics2DSK; // SKCanvasElement
using Microsoft.UI.Xaml;
using Windows.Foundation; // Size

public sealed class PlotCanvas : SKCanvasElement
{
    public PlotRenderer? Renderer { get; set; }

    public PlotCanvas()
    {
        if (!IsSupportedOnCurrentPlatform())
        {
            // Skia rendering is required. On unsupported targets, show a fallback
            // in XAML instead of constructing this element.
            throw new PlatformNotSupportedException(
                "SKCanvasElement requires Skia rendering on the current platform.");
        }
    }

    // origin is already translated so 0,0 is the element's top-left.
    // Drawing outside 'area' is clipped.
    protected override void RenderOverride(SKCanvas canvas, Size area)
        => Renderer?.Render(canvas, (float)area.Width, (float)area.Height);

    public void RequestRedraw() => Invalidate(); // triggers RenderOverride
}
```

```csharp
public sealed class PlotRenderer : IDisposable
{
    // Reused across frames. Never allocate paints inside Render.
    private readonly SKPaint _stroke = new()
    {
        Color = SKColors.SteelBlue,
        StrokeWidth = 2,
        IsAntialias = true,
        Style = SKPaintStyle.Stroke,
    };

    private float _phase;

    public void Advance(float dt) => _phase += dt; // mutate state off the render thread

    public void Render(SKCanvas canvas, float width, float height)
    {
        canvas.Clear(SKColors.Transparent);
        using var path = new SKPath();
        for (float x = 0; x <= width; x += 4)
        {
            var y = height / 2 + MathF.Sin(x * 0.03f + _phase) * height * 0.3f;
            if (x == 0) path.MoveTo(x, y); else path.LineTo(x, y);
        }
        canvas.DrawPath(path, _stroke);
    }

    public void Dispose() => _stroke.Dispose();
}
```

XAML. `SKCanvasElement` is a `FrameworkElement`, so it stretches and lays out like any other:

```xml
<Grid>
    <local:PlotCanvas x:Name="Plot"
                      HorizontalAlignment="Stretch"
                      VerticalAlignment="Stretch" />
</Grid>
```

Wire-up and animation. `RenderOverride` is cached and only re-runs when `Invalidate()` is called, so drive animation from a timer (or a composition frame callback) on the UI thread:

```csharp
public sealed partial class PlotPage : Page
{
    private readonly PlotRenderer _renderer = new();
    private long _lastTicks;

    public PlotPage()
    {
        InitializeComponent();
        Plot.Renderer = _renderer;

        var timer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(16) };
        timer.Tick += (_, _) =>
        {
            var now = Environment.TickCount64;
            var dt = _lastTicks == 0 ? 0 : (now - _lastTicks) / 1000f;
            _lastTicks = now;
            _renderer.Advance(dt);
            Plot.RequestRedraw();
        };
        timer.Start();

        Unloaded += (_, _) => { timer.Stop(); _renderer.Dispose(); };
    }
}
```

## Controlling the drawing size

`SKCanvasElement` sizes by normal layout. To make the drawing surface a specific size, set `Width`/`Height` or override `MeasureOverride`/`ArrangeOverride` on the element. The `area` passed to `RenderOverride` reflects the arranged size.

## Static method and platform support

```csharp
public static bool IsSupportedOnCurrentPlatform();
```

Returns true on `netX.0-desktop` and on other targets using Skia rendering (for example `netX.0-android` with the `SkiaRenderer` feature). Use it to choose between the canvas element and a fallback UI rather than crashing on unsupported targets.

## SKXamlCanvas (fallback / porting path)

```csharp
using SkiaSharp;
using SkiaSharp.Views.Windows; // SKPaintSurfaceEventArgs on Uno.WinUI

private void OnPaintSurface(object? sender, SKPaintSurfaceEventArgs e)
{
    var canvas = e.Surface.Canvas;
    canvas.Clear(SKColors.White);
    // existing draw code ports across unchanged
}
```

```xml
xmlns:skia="using:SkiaSharp.Views.Windows"
<skia:SKXamlCanvas x:Name="Canvas" PaintSurface="OnPaintSurface" />
```

Call `Canvas.Invalidate()` to repaint. Prefer migrating to `SKCanvasElement` on Skia targets once the port is stable.

## WinAppSDK specifics

When running `SKCanvasElement` on WinAppSDK, build with an `x64` or `ARM64` solution configuration (not `AnyCPU`): create the configuration in Configuration Manager, assign it to the Uno project, and debug with the configuration matching your machine.

## Full sample

The Uno.Samples repository has a complete `SKCanvasElement` sample referenced from the official control docs: https://platform.uno/docs/articles/controls/SKCanvasElement.html
