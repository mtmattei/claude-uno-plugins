# Rendering & Composition Skills - WinUI 3 XAML

## RENDERING-001: Use Composition APIs for Animations

**Rule**: Use Windows.UI.Composition APIs instead of XAML Storyboards for smooth, 60 FPS animations.

**Why**: Composition animations run on a separate compositor thread, independent of the UI thread. They continue smoothly even when the UI thread is busy. XAML Storyboards run on the UI thread.

**Example (Correct)**:
```csharp
private void AnimateElement(UIElement element)
{
    var visual = ElementCompositionPreview.GetElementVisual(element);
    var compositor = visual.Compositor;

    // Fade animation
    var fadeAnimation = compositor.CreateScalarKeyFrameAnimation();
    fadeAnimation.InsertKeyFrame(0f, 0f);
    fadeAnimation.InsertKeyFrame(1f, 1f);
    fadeAnimation.Duration = TimeSpan.FromMilliseconds(300);

    // Slide animation
    var slideAnimation = compositor.CreateVector3KeyFrameAnimation();
    slideAnimation.InsertKeyFrame(0f, new Vector3(0, 50, 0));
    slideAnimation.InsertKeyFrame(1f, new Vector3(0, 0, 0));
    slideAnimation.Duration = TimeSpan.FromMilliseconds(300);

    // Custom easing
    var easing = compositor.CreateCubicBezierEasingFunction(
        new Vector2(0.1f, 0.9f),
        new Vector2(0.2f, 1.0f));
    slideAnimation.InsertKeyFrame(1f, Vector3.Zero, easing);

    // Start animations
    visual.StartAnimation("Opacity", fadeAnimation);
    visual.StartAnimation("Offset", slideAnimation);
}
```

**Animatable Composition Properties**:
- `Opacity` - Transparency (0-1)
- `Offset` - Position (Vector3)
- `Scale` - Size scaling (Vector3)
- `RotationAngle` - Rotation in radians
- `RotationAngleInDegrees` - Rotation in degrees
- `CenterPoint` - Transform origin

**Common Mistakes**:
```csharp
// WRONG: XAML Storyboard runs on UI thread
var storyboard = new Storyboard();
var animation = new DoubleAnimation
{
    From = 0, To = 1,
    Duration = TimeSpan.FromMilliseconds(300)
};
Storyboard.SetTarget(animation, element);
Storyboard.SetTargetProperty(animation, "Opacity");
storyboard.Begin(); // Blocks UI thread
```

**Uno Platform Notes**: Composition APIs are supported on Skia backends. Some effects require GPU rendering. Check `CompositionCapabilities.AreEffectsSupported()`.

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/composition/composition-animation

---

## RENDERING-002: Use Visibility Instead of Opacity=0

**Rule**: To hide elements, use `Visibility.Collapsed` instead of `Opacity="0"`. For elements that may not be needed, use `x:Load="False"`.

**Why**:
- `Opacity=0`: Element is still rendered, participates in layout, receives hit tests
- `Visibility.Collapsed`: Skips rendering and layout, no hit testing
- `x:Load=false`: Element not created at all, memory released

**Example (Correct)**:
```xml
<!-- Toggle visibility for show/hide -->
<Grid Visibility="{x:Bind ViewModel.IsVisible, Mode=OneWay}"/>

<!-- Deferred loading for conditional UI -->
<Grid x:Name="AdvancedOptions"
      x:Load="{x:Bind ViewModel.ShowAdvanced, Mode=OneWay}">
    <!-- Complex content not created until ShowAdvanced is true -->
</Grid>
```

**When to use each**:
| Scenario | Use |
|----------|-----|
| Fade animation | Opacity |
| Toggle show/hide (keep in memory) | Visibility |
| Conditional UI (release memory) | x:Load |

**Common Mistakes**:
```xml
<!-- WRONG: Opacity=0 still renders and hit-tests -->
<Grid Opacity="{x:Bind helpers:Converters.BoolToOpacity(IsVisible)}">
    <ExpensiveControl/>
</Grid>
```

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.uielement.visibility

---

## RENDERING-003: Limit Blur and Acrylic Effects

**Rule**: Use AcrylicBrush and blur effects sparingly. Limit to small, static surfaces. Disable during scrolling/animations.

**Why**: Blur effects are GPU-intensive. Full-screen acrylic during scrolling can drop frame rates to 30 FPS. Use solid colors during motion.

**Example (Correct)**:
```xml
<!-- Acrylic on small, static surfaces only -->
<Grid>
    <!-- Header with acrylic (small, doesn't scroll) -->
    <Border Height="48" VerticalAlignment="Top">
        <Border.Background>
            <AcrylicBrush TintColor="{ThemeResource SystemAccentColor}"
                          TintOpacity="0.8"
                          FallbackColor="{ThemeResource SystemAccentColor}"/>
        </Border.Background>
    </Border>

    <!-- Main content with solid background -->
    <ScrollViewer Margin="0,48,0,0">
        <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
            <!-- Content -->
        </Grid>
    </ScrollViewer>
</Grid>
```

**Disable effects during animation**:
```csharp
private void OnScrollViewerViewChanging(object sender, ScrollViewerViewChangingEventArgs e)
{
    // Switch to solid during scroll
    HeaderBorder.Background = _solidBrush;
}

private void OnScrollViewerViewChanged(object sender, ScrollViewerViewChangedEventArgs e)
{
    if (!e.IsIntermediate)
    {
        // Restore acrylic after scroll completes
        HeaderBorder.Background = _acrylicBrush;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/acrylic

---

## RENDERING-004: Use ThemeShadow for Elevation

**Rule**: Use `ThemeShadow` with `Translation` Z-value for elevation effects instead of custom drop shadows on every element.

**Why**: ThemeShadow is optimized and provides consistent elevation across controls. Custom DropShadow on many elements is expensive.

**Example (Correct)**:
```xml
<!-- Card with elevation using ThemeShadow -->
<Grid>
    <Grid.Resources>
        <ThemeShadow x:Name="SharedShadow"/>
    </Grid.Resources>

    <Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
            CornerRadius="8"
            Shadow="{StaticResource SharedShadow}"
            Translation="0,0,32">
        <StackPanel Padding="16">
            <TextBlock Text="Card Title" Style="{StaticResource SubtitleTextBlockStyle}"/>
            <TextBlock Text="Card content goes here"/>
        </StackPanel>
    </Border>
</Grid>
```

**Common Mistakes**:
```csharp
// WRONG: Custom DropShadow on every card
foreach (var card in Cards)
{
    var shadow = compositor.CreateDropShadow();
    shadow.BlurRadius = 32f; // Expensive blur
    shadow.Offset = new Vector3(0, 8, 0);
    // Apply to card...
}
```

**Uno Platform Notes**: `ThemeShadow` is supported on Skia backends. Use `Translation` property with Z > 0 to enable shadows.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/depth-shadow

---

## RENDERING-005: Cache Static Complex Visuals

**Rule**: For complex static visuals (vector graphics, rich text), consider using `CacheMode="BitmapCache"` to rasterize once.

**Why**: Complex vector paths and rich formatting are re-rendered every frame. BitmapCache renders once and reuses the bitmap. Trade-off: memory for CPU.

**Example (Correct)**:
```xml
<!-- Complex static logo - cache as bitmap -->
<Canvas CacheMode="BitmapCache">
    <Path Data="M 0,0 L 100,50 L 50,100 Z" Fill="Blue"/>
    <Path Data="M 10,10 C 20,20 40,20 50,10" Stroke="Red" StrokeThickness="2"/>
    <!-- Many more complex paths -->
</Canvas>
```

```csharp
// Invalidate cache when content changes
private void UpdateLogo()
{
    LogoCanvas.CacheMode = null; // Clear cache
    // Update paths...
    LogoCanvas.CacheMode = new BitmapCache(); // Re-cache
}
```

**When to use BitmapCache**:
- Static complex vector graphics
- Logos and icons with many paths
- Rich formatted text that doesn't change

**When NOT to use**:
- Animated content
- Frequently changing content
- Simple elements (adds overhead)

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.media.bitmapcache

---

## RENDERING-006: Optimize Image Loading

**Rule**: Set `DecodePixelWidth`/`DecodePixelHeight` on BitmapImage to decode at display size, not full resolution.

**Why**: A 4000x3000 photo displayed at 400x300 wastes 10x memory if decoded at full size. Decode at display size to reduce memory and improve performance.

**Example (Correct)**:
```xml
<Image Width="200" Height="200">
    <Image.Source>
        <BitmapImage UriSource="{x:Bind PhotoUrl}"
                     DecodePixelWidth="200"
                     DecodePixelType="Logical"/>
    </Image.Source>
</Image>
```

```csharp
// Programmatic image loading with decode size
var bitmap = new BitmapImage
{
    DecodePixelWidth = 400,
    DecodePixelType = DecodePixelType.Logical
};
await bitmap.SetSourceAsync(stream);
MyImage.Source = bitmap;
```

**Common Mistakes**:
```xml
<!-- WRONG: Full resolution decode for thumbnail -->
<Image Width="100" Height="100" Source="{x:Bind HighResPhotoUrl}"/>
```

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.media.imaging.bitmapimage.decodepixelwidth

---

## RENDERING-007: Use ConnectedAnimation for Page Transitions

**Rule**: Use `ConnectedAnimationService` for element continuity between pages to create polished transitions.

**Why**: ConnectedAnimation provides smooth, system-consistent transitions that give users spatial context when navigating.

**Example (Correct)**:
```csharp
// Source page - prepare animation before navigation
private void ListView_ItemClick(object sender, ItemClickEventArgs e)
{
    var item = e.ClickedItem as Photo;

    // Prepare connected animation
    ListView.PrepareConnectedAnimation("photoAnimation", item, "PhotoImage");

    Frame.Navigate(typeof(DetailPage), item);
}

// Destination page - start animation
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);

    var photo = e.Parameter as Photo;
    ViewModel.Photo = photo;

    var animation = ConnectedAnimationService.GetForCurrentView()
        .GetAnimation("photoAnimation");

    if (animation != null)
    {
        animation.TryStart(DetailImage);
    }
}

// Back navigation - animate back
protected override void OnNavigatingFrom(NavigatingCancelEventArgs e)
{
    if (e.NavigationMode == NavigationMode.Back)
    {
        ConnectedAnimationService.GetForCurrentView()
            .PrepareToAnimate("photoAnimation", DetailImage);
    }
    base.OnNavigatingFrom(e);
}
```

**Uno Platform Notes**: ConnectedAnimation support varies by platform. Test on target platforms.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/motion/connected-animation
