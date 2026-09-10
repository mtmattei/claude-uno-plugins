# Custom Controls & Advanced UI (Medium Priority)

## SkiaSharp Custom Control Pattern

**When to apply:** Custom rendering, 2D/3D graphics

### SKCanvasElement Control
```csharp
using SkiaSharp;
using SkiaSharp.Views.Windows;
using Microsoft.UI.Xaml;

public sealed class MyCanvasElement : SKCanvasElement
{
    private MyRenderer? _renderer;

    public MyRenderer? Renderer
    {
        get => _renderer;
        set => _renderer = value;
    }

    public MyCanvasElement()
    {
        if (!IsSupportedOnCurrentPlatform())
        {
            throw new PlatformNotSupportedException("SKCanvasElement only supported on Skia platforms.");
        }
    }

    protected override void RenderOverride(SKCanvas canvas, Size area)
    {
        if (_renderer is null)
        {
            return;
        }

        _renderer.Render(canvas, (int)area.Width, (int)area.Height);
    }

    public void RequestInvalidate()
    {
        Invalidate();
    }
}
```

### Renderer Pattern (Separation of Concerns)
```csharp
public class MyRenderer : IDisposable
{
    private readonly SKPaint _paint;
    private List<Point> _points = new();

    public MyRenderer()
    {
        _paint = new SKPaint
        {
            Color = SKColors.Blue,
            StrokeWidth = 2,
            IsAntialias = true,
            Style = SKPaintStyle.Fill
        };
    }

    public void Render(SKCanvas canvas, int width, int height)
    {
        // Clear
        canvas.Clear(SKColors.White);

        // Render content
        foreach (var point in _points)
        {
            canvas.DrawCircle((float)point.X, (float)point.Y, 5, _paint);
        }
    }

    public void AddPoint(Point point)
    {
        _points.Add(point);
    }

    public void Dispose()
    {
        _paint?.Dispose();
    }
}
```

### XAML Usage
```xml
<local:MyCanvasElement x:Name="Canvas"
                       HorizontalAlignment="Stretch"
                       VerticalAlignment="Stretch" />
```

### Code-Behind Integration
```csharp
public sealed partial class MainPage : Page
{
    private readonly MyRenderer _renderer;

    public MainPage()
    {
        this.InitializeComponent();
        _renderer = new MyRenderer();
        Canvas.Renderer = _renderer;

        // Animation loop
        var timer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(16) };
        timer.Tick += (s, e) => {
            _renderer.Update();
            Canvas.RequestInvalidate();
        };
        timer.Start();
    }
}
```

### Performance Tips
- Reuse SKPaint instances (don't create per-frame)
- Pre-calculate values in Update(), render in Render()
- Use depth sorting for 3D projections
- Dispose SKPaint objects when done

### References
- Example: C:\Users\Platform006\source\repos\FibonacciSphere\FibonacciSphere\Controls\SphereCanvasElement.cs
- Example: C:\Users\Platform006\source\repos\FibonacciSphere\FibonacciSphere\Rendering\SphereRenderer.cs

## ItemsControl-Based Custom Control

**When to apply:** List-based custom controls

### UserControl Pattern
```csharp
public sealed partial class HorizontalCalendar : UserControl, INotifyPropertyChanged
{
    private ObservableCollection<CalendarDate> _dates = new();
    private CalendarDate? _selectedDate;
    private DispatcherTimer? _midnightTimer;

    public ObservableCollection<CalendarDate> Dates
    {
        get => _dates;
        set { _dates = value; OnPropertyChanged(); }
    }

    public string DisplayMonth => CurrentMonth.ToString("MMMM yyyy");

    public event EventHandler<CalendarDate>? DateSelected;

    public HorizontalCalendar()
    {
        this.InitializeComponent();
        InitializeDates();
        SetupMidnightTimer();
    }

    private void InitializeDates()
    {
        var today = DateTime.Today;
        for (int i = -30; i <= 30; i++)
        {
            var date = today.AddDays(i);
            Dates.Add(new CalendarDate
            {
                Date = date,
                DayName = date.ToString("ddd"),
                DayNumber = date.Day.ToString(),
                IsToday = date.Date == today.Date,
                IsSelected = false
            });
        }
    }

    public void SelectDate(DateTime date)
    {
        var targetDate = Dates.FirstOrDefault(d => d.Date.Date == date.Date);
        if (targetDate == null) return;

        if (_selectedDate != null)
        {
            _selectedDate.IsSelected = false;
        }

        targetDate.IsSelected = true;
        _selectedDate = targetDate;

        OnPropertyChanged(nameof(SelectedDateText));
        ScrollToDate(targetDate, animate: true);
        DateSelected?.Invoke(this, targetDate);
    }

    private void SetupMidnightTimer()
    {
        var now = DateTime.Now;
        var tomorrow = now.Date.AddDays(1);
        var timeUntilMidnight = tomorrow - now;

        _midnightTimer = new DispatcherTimer
        {
            Interval = timeUntilMidnight
        };
        _midnightTimer.Tick += OnMidnightTimerTick;
        _midnightTimer.Start();
    }

    private void OnMidnightTimerTick(object? sender, object e)
    {
        RefreshTodayHighlight();
        _midnightTimer?.Stop();
        SetupMidnightTimer();
    }

    public event PropertyChangedEventHandler? PropertyChanged;

    private void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

### XAML Template
```xml
<UserControl x:Class="MyApp.Controls.HorizontalCalendar"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <Grid>
        <ScrollViewer HorizontalScrollBarVisibility="Auto"
                      VerticalScrollBarVisibility="Disabled">

            <ItemsRepeater ItemsSource="{x:Bind Dates, Mode=OneWay}">
                <ItemsRepeater.Layout>
                    <StackLayout Orientation="Horizontal" Spacing="8" />
                </ItemsRepeater.Layout>

                <ItemsRepeater.ItemTemplate>
                    <DataTemplate x:DataType="models:CalendarDate">
                        <Border Width="60" Height="80"
                                CornerRadius="12"
                                Background="{x:Bind IsSelected, Converter={StaticResource SelectedBackgroundConverter}, Mode=OneWay}"
                                BorderBrush="{x:Bind IsToday, Converter={StaticResource TodayBorderConverter}, Mode=OneWay}"
                                BorderThickness="2"
                                Tapped="DateTapped">

                            <StackPanel VerticalAlignment="Center" Spacing="4">
                                <TextBlock Text="{x:Bind DayName}"
                                          FontSize="12"
                                          HorizontalAlignment="Center" />
                                <TextBlock Text="{x:Bind DayNumber}"
                                          FontSize="20"
                                          FontWeight="Bold"
                                          HorizontalAlignment="Center" />
                            </StackPanel>

                        </Border>
                    </DataTemplate>
                </ItemsRepeater.ItemTemplate>

            </ItemsRepeater>

        </ScrollViewer>
    </Grid>

</UserControl>
```

### Key Patterns
- UserControl with INotifyPropertyChanged for internal state
- ObservableCollection for reactive updates
- Event-based interaction (DateSelected event)
- DispatcherTimer for time-based logic
- ItemsRepeater for performance

### References
- Example: C:\Users\Platform006\source\repos\HorizontalCalendar\HorizontalCalendar\Controls\HorizontalCalendar.xaml.cs

## Lottie Animations

**When to apply:** Animated icons, loading indicators, splash screens

### Setup
```xml
<UnoFeatures>
  Lottie;
  <!-- other features -->
</UnoFeatures>
```

### XAML Usage
```xml
xmlns:lottie="using:Microsoft.UI.Xaml.Controls"
xmlns:winui="using:Microsoft.UI.Xaml.Controls.AnimatedVisuals"

<winui:AnimatedVisualPlayer AutoPlay="True" Loop="True">
    <lottie:LottieVisualSource UriSource="ms-appx:///Assets/loading.json" />
</winui:AnimatedVisualPlayer>
```

### Splash Screen with Lottie
```xml
<utu:ExtendedSplashScreen x:Name="Splash">
    <utu:ExtendedSplashScreen.LoadingContent>
        <Grid Background="{ThemeResource BackgroundColor}">
            <winui:AnimatedVisualPlayer AutoPlay="True">
                <lottie:LottieVisualSource UriSource="ms-appx:///Assets/SplashAnimation.json" />
            </winui:AnimatedVisualPlayer>
        </Grid>
    </utu:ExtendedSplashScreen.LoadingContent>

    <utu:ExtendedSplashScreen.Content>
        <Frame x:Name="ShellFrame" />
    </utu:ExtendedSplashScreen.Content>
</utu:ExtendedSplashScreen>
```

### References
- Official docs: https://platform.uno/docs/articles/features/Lottie.html
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\Shell.xaml

## Responsive Image Display

**When to apply:** Photos, hero images, thumbnails

### Image Stretch Modes
```xml
<!-- For photographs and hero images -->
<Image Source="{x:Bind ViewModel.PhotoUrl}"
       Stretch="UniformToFill"
       HorizontalAlignment="Center"
       VerticalAlignment="Center" />

<!-- For icons and logos -->
<Image Source="{x:Bind ViewModel.LogoUrl}"
       Stretch="Uniform"
       MaxWidth="100"
       MaxHeight="100" />

<!-- Background images -->
<Grid>
    <Grid.Background>
        <ImageBrush ImageSource="ms-appx:///Assets/background.jpg"
                    Stretch="UniformToFill" />
    </Grid.Background>
</Grid>
```

### Placeholder Images
```xml
<!-- When no image is available -->
<Image Source="https://picsum.photos/400/300"
       Stretch="UniformToFill" />
```

### FlipView for Image Galleries
```xml
<FlipView ItemsSource="{x:Bind ViewModel.Images}">
    <FlipView.ItemTemplate>
        <DataTemplate x:DataType="x:String">
            <Image Source="{x:Bind}"
                   Stretch="Uniform"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center" />
        </DataTemplate>
    </FlipView.ItemTemplate>
</FlipView>
```

### References
- Pattern from HockeyBarn and other sample apps
