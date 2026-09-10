# Settings, Theme, DI, and Services Reference

Patterns for migrating WPF settings, theme switching, and static singletons to Uno Platform's DI-based architecture with IWritableOptions, SystemThemeHelper, and service interfaces.

---

## Settings Migration

### WPF Pattern (Before)

```csharp
// WPF — Properties.Settings.Default
Properties.Settings.Default.ShowConfirmation = true;
Properties.Settings.Default.FontSize = 14;
Properties.Settings.Default.Save();

// Reading
var fontSize = Properties.Settings.Default.FontSize;
```

### Uno Platform Pattern (After)

**Step 1: Define an immutable settings record**

```csharp
public partial record AppSettings
{
    public bool ShowConfirmation { get; init; } = true;
    public int FontSize { get; init; } = 14;
    public bool FirstRun { get; init; } = true;
    public string Theme { get; init; } = "System";
}
```

**Step 2: Register in App.xaml.cs host configuration**

```csharp
.UseConfiguration(configure: configBuilder =>
    configBuilder
        .EmbeddedSource<App>()
        .Section<AppSettings>()
)
```

`Section<AppSettings>()` registers **both** `IOptions<AppSettings>` and `IWritableOptions<AppSettings>` — no extra registration needed.

**Step 3: Create appsettings.json as EmbeddedResource**

```json
{
    "AppSettings": {
        "ShowConfirmation": true,
        "FontSize": 14,
        "FirstRun": true,
        "Theme": "System"
    }
}
```

In `.csproj`:
```xml
<ItemGroup>
    <EmbeddedResource Include="appsettings.json" />
</ItemGroup>
```

**Step 4: Use in MVUX Model**

```csharp
public partial record GeneralSettingsModel(IWritableOptions<AppSettings> Settings)
{
    public IState<bool> ShowConfirmation => State.Async(async ct =>
    {
        var s = await Settings.GetAsync(ct);
        return s.ShowConfirmation;
    });

    public IState<int> FontSize => State.Async(async ct =>
    {
        var s = await Settings.GetAsync(ct);
        return s.FontSize;
    });

    public async ValueTask ToggleShowConfirmation(bool value) =>
        await Settings.UpdateAsync(s => s with { ShowConfirmation = value });

    public async ValueTask SetFontSize(int value) =>
        await Settings.UpdateAsync(s => s with { FontSize = value });
}
```

### ToggleSwitch Initialization Guard

When binding ToggleSwitch to settings, the `Toggled` event fires during initialization. Use a loading guard:

```csharp
// In code-behind (or handle in Model)
private bool _isLoading = true;

private void Page_Loaded(object sender, RoutedEventArgs e)
{
    _isLoading = false;
}

private async void ToggleSwitch_Toggled(object sender, RoutedEventArgs e)
{
    if (_isLoading) return;
    // Handle toggle
}
```

### Namespace Collision Warning

If your project namespace starts with a prefix that shadows Uno namespaces (e.g., `MyApp.Uno`), you must use the `global::` prefix:

```csharp
// ERROR: MyApp.Uno.Extensions.Configuration is not what you want
IWritableOptions<AppSettings> settings;

// FIX: Explicit global prefix
global::Uno.Extensions.Configuration.IWritableOptions<AppSettings> settings;
```

This applies to any `Uno.*` type when your project namespace starts with something that contains `Uno`.

---

## Theme Switching

### WPF Pattern (Before)

```csharp
// WPF
App.SetTheme(isDark ? ApplicationTheme.Dark : ApplicationTheme.Light);
```

### Uno Platform Pattern (After)

```csharp
// In ShellPage.Loaded — apply saved theme
private async void ShellPage_Loaded(object sender, RoutedEventArgs e)
{
    var settings = App.Host.Services.GetRequiredService<IOptions<AppSettings>>();
    var theme = settings.Value.Theme switch
    {
        "Dark" => ElementTheme.Dark,
        "Light" => ElementTheme.Light,
        _ => ElementTheme.Default
    };
    global::Uno.Toolkit.UI.SystemThemeHelper.SetApplicationTheme(this.XamlRoot, theme);
}
```

**Key points:**
- Must use `global::Uno.Toolkit.UI.SystemThemeHelper` (namespace collision risk)
- Apply in `ShellPage.Loaded`, not `App.OnLaunched` (XamlRoot not available yet)
- Uno already has `Uno.Extensions.Toolkit.IThemeService` — don't create a custom one (name collision)
- `ElementTheme.Default` follows system theme

---

## Singleton → DI Migration

### WPF Pattern (Before)

```csharp
// WPF — static singletons
public class Singleton<T> where T : new()
{
    private static T _instance;
    public static T Instance => _instance ??= new T();
}

// Usage
var history = Singleton<HistoryService>.Instance;
history.AddEntry(text);
```

### Uno Platform Pattern (After)

**Step 1: Define an interface**

```csharp
public interface IHistoryService
{
    Task<IReadOnlyList<HistoryInfo>> GetEntriesAsync(CancellationToken ct = default);
    Task AddEntryAsync(HistoryInfo entry, CancellationToken ct = default);
    Task ClearAsync(CancellationToken ct = default);
}
```

**Step 2: Implement**

```csharp
public class FileHistoryService : IHistoryService
{
    private readonly string _filePath;

    public FileHistoryService()
    {
        var folder = ApplicationData.Current.LocalFolder.Path;
        _filePath = Path.Combine(folder, "history.json");
    }

    public async Task<IReadOnlyList<HistoryInfo>> GetEntriesAsync(CancellationToken ct)
    {
        if (!File.Exists(_filePath)) return Array.Empty<HistoryInfo>();
        var json = await File.ReadAllTextAsync(_filePath, ct);
        return JsonSerializer.Deserialize<List<HistoryInfo>>(json) ?? [];
    }

    // ...
}
```

**Step 3: Register in DI**

```csharp
// In App.xaml.cs ConfigureServices
services.AddSingleton<IHistoryService, FileHistoryService>();
```

**Step 4: Consume via MVUX Model constructor injection**

```csharp
public partial record HistoryModel(IHistoryService HistoryService)
{
    public IListFeed<HistoryInfo> Entries => ListFeed.Async(
        async ct => await HistoryService.GetEntriesAsync(ct)
    );
}
```

---

## Service Abstraction Pattern (OCR Engine Example)

For platform-specific services, define a common interface and register platform-specific implementations.

### Interface

```csharp
public interface IOcrEngine
{
    bool IsAvailable { get; }
    Task<IOcrResult> RecognizeAsync(Stream imageStream, string language, CancellationToken ct);
}

public interface IOcrResult
{
    IReadOnlyList<IOcrLine> Lines { get; }
    string FullText { get; }
}
```

### Platform-Specific Implementation

```csharp
// In Platforms/Windows/WindowsOcrEngine.cs
#if WINDOWS
public class WindowsOcrEngine : IOcrEngine
{
    public bool IsAvailable => true;

    public async Task<IOcrResult> RecognizeAsync(Stream imageStream, string language, CancellationToken ct)
    {
        // Use Windows.Media.Ocr.OcrEngine
        var decoder = await BitmapDecoder.CreateAsync(imageStream.AsRandomAccessStream());
        var bitmap = await decoder.GetSoftwareBitmapAsync();
        var engine = Windows.Media.Ocr.OcrEngine.TryCreateFromLanguage(new Language(language));
        var result = await engine.RecognizeAsync(bitmap);
        return new WindowsOcrResult(result);
    }
}
#endif
```

### Registration with Platform Guards

```csharp
// In App.xaml.cs
private static void ConfigureServices(HostBuilderContext context, IServiceCollection services)
{
    #if WINDOWS
    services.AddSingleton<IOcrEngine, WindowsOcrEngine>();
    #else
    services.AddSingleton<IOcrEngine, NullOcrEngine>(); // No-op on non-Windows
    #endif

    services.AddSingleton<IOcrService, OcrService>(); // Facade
}
```

### Facade Pattern

```csharp
public class OcrService : IOcrService
{
    private readonly IEnumerable<IOcrEngine> _engines;

    public OcrService(IEnumerable<IOcrEngine> engines)
    {
        _engines = engines;
    }

    public async Task<IOcrResult> RecognizeAsync(Stream image, string language, CancellationToken ct)
    {
        var engine = _engines.FirstOrDefault(e => e.IsAvailable)
            ?? throw new InvalidOperationException("No OCR engine available on this platform");
        return await engine.RecognizeAsync(image, language, ct);
    }
}
```

### Service Boundary Rule

**Never pass `SoftwareBitmap` across service boundaries** — it is Windows-only. Use `Stream` or `byte[]` as the cross-platform currency at interface boundaries. Convert to platform-specific types inside the platform-specific implementation.

---

## App.Host Accessibility

The Uno template makes `App.Host` `protected` by default. If Shell or ShellPage need access to the DI container:

```csharp
// In App.xaml.cs — change from protected to public
public IHost Host { get; private set; }
```

This allows `App.Host.Services.GetRequiredService<T>()` from Shell code-behind when needed (e.g., theme application in Loaded handler).

---

## Language Abstraction

WPF apps using `Windows.Globalization.Language` need a cross-platform wrapper:

```csharp
// WPF: Windows.Globalization.Language (Windows-only)
// Uno: Custom GlobalLang wrapping CultureInfo

public record GlobalLang(CultureInfo Culture)
{
    public string DisplayName => Culture.DisplayName;
    public string LanguageTag => Culture.Name;
    public LanguageLayoutDirection LayoutDirection =>
        Culture.TextInfo.IsRightToLeft
            ? LanguageLayoutDirection.Rtl
            : LanguageLayoutDirection.Ltr;
}

// Replace: InputLanguageManager.Current.CurrentInputLanguage
// With:    CultureInfo.CurrentCulture
```

---

## Common Mistakes

- Forgetting that `Section<AppSettings>()` registers both `IOptions<T>` and `IWritableOptions<T>`
- Not guarding ToggleSwitch.Toggled events during initialization (fires before user interaction)
- Creating a custom `IThemeService` that conflicts with Uno's built-in `Uno.Extensions.Toolkit.IThemeService`
- Applying theme in `App.OnLaunched` instead of `ShellPage.Loaded` (XamlRoot not ready)
- Passing `SoftwareBitmap` in service interfaces (breaks on non-Windows)
- Not using `global::` prefix when project namespace shadows `Uno.*`
- Keeping `Singleton<T>.Instance` alongside DI (inconsistent service resolution)
