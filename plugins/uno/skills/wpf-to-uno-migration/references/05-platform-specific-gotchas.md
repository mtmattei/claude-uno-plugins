# Platform-Specific Patterns and Gotchas Reference

Handling Windows-only features, ContentDialog conversions, first-run flows, in-app notifications, and the complete runtime gotchas table.

---

## Platform-Specific UI in XAML

**You cannot use `#if WINDOWS` preprocessor directives in XAML.** Use this two-step pattern instead:

### Step 1: Hide in XAML

```xml
<!-- Set Collapsed by default — code-behind will show on Windows -->
<StackPanel x:Name="WindowsOnlySection" Visibility="Collapsed">
    <TextBlock Text="Windows-specific feature" />
    <Button Content="Open in Explorer" Click="OpenExplorer_Click" />
</StackPanel>

<!-- Show info text on non-Windows -->
<TextBlock x:Name="NonWindowsInfo"
           Text="This feature is only available on Windows"
           Visibility="Visible" />
```

### Step 2: Toggle in Code-Behind

```csharp
protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);

    #if WINDOWS
    WindowsOnlySection.Visibility = Visibility.Visible;
    NonWindowsInfo.Visibility = Visibility.Collapsed;
    #endif
}
```

### Windows-Only Features Requiring Guards

| Feature | Guard Needed | Alternative on Other Platforms |
|---|---|---|
| Startup task (launch at login) | `#if WINDOWS` | Not available; hide UI |
| Global hotkeys | `#if WINDOWS` | Not available; hide UI |
| File Explorer integration | `#if WINDOWS` | Use platform file picker |
| `Process.Start()` | `#if WINDOWS` | Not available; show info text |
| System tray icon | `#if WINDOWS` | Not available |
| Registry access | `#if WINDOWS` | Use `IWritableOptions<T>` |
| COM interop | `#if WINDOWS` | Rewrite with cross-platform API |
| `Application.Exit()` | `#if WINDOWS` | Not implemented on Wasm (acceptable) |

---

## First-Run Flow

### WPF Pattern (Before)

```csharp
// WPF — standalone FirstRunWindow
if (Settings.Default.FirstRun)
{
    var firstRun = new FirstRunWindow();
    firstRun.ShowDialog();
    Settings.Default.FirstRun = false;
    Settings.Default.Save();
}
```

### Uno Platform Pattern (After)

**Use ContentDialog in ShellPage.Loaded — NOT route-based navigation.**

Route-based first-run was tried and abandoned: root-level sibling routes can't navigate back to Shell properly, creating a broken back-stack.

```csharp
// In ShellPage.xaml.cs
private async void ShellPage_Loaded(object sender, RoutedEventArgs e)
{
    // Apply saved theme first
    ApplySavedTheme();

    // Check first-run
    var settings = App.Host.Services.GetRequiredService<IWritableOptions<AppSettings>>();
    var current = await settings.GetAsync(CancellationToken.None);

    if (current.FirstRun)
    {
        var dialog = new FirstRunDialog
        {
            XamlRoot = this.XamlRoot
        };
        await dialog.ShowAsync();
        await settings.UpdateAsync(s => s with { FirstRun = false });
    }
}
```

---

## In-App Notifications

### WPF Pattern (Before)

```csharp
// WPF — Windows toast notifications
new ToastContentBuilder()
    .AddText("OCR Complete")
    .AddText("Text copied to clipboard")
    .Show();
```

### Uno Platform Pattern (After)

Use `InfoBar` overlay in ShellPage for cross-platform notifications:

```csharp
public interface IInAppNotificationService
{
    void Show(string title, string message, InfoBarSeverity severity = InfoBarSeverity.Informational);
}

public class InAppNotificationService : IInAppNotificationService
{
    private Panel _host;
    private const int AutoDismissMs = 4000;

    public void SetHost(Panel host) => _host = host;

    public void Show(string title, string message, InfoBarSeverity severity)
    {
        var infoBar = new InfoBar
        {
            Title = title,
            Message = message,
            Severity = severity,
            IsOpen = true
        };

        _host.Children.Add(infoBar);

        var timer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(AutoDismissMs) };
        timer.Tick += (s, e) =>
        {
            timer.Stop();
            infoBar.IsOpen = false;
            _host.Children.Remove(infoBar);
        };
        timer.Start();
    }
}
```

### Shell XAML Setup

```xml
<!-- In ShellPage.xaml — overlay on top of NavigationView -->
<Grid>
    <NavigationView uen:Region.Attached="true">
        <!-- ... menu items and Frame ... -->
    </NavigationView>

    <!-- Notification overlay -->
    <StackPanel x:Name="NotificationHost"
                VerticalAlignment="Top"
                HorizontalAlignment="Center"
                Margin="0,48,0,0" />
</Grid>
```

```csharp
// In ShellPage.Loaded
var notificationService = App.Host.Services.GetRequiredService<IInAppNotificationService>();
(notificationService as InAppNotificationService)?.SetHost(NotificationHost);
```

Register as singleton in DI:
```csharp
services.AddSingleton<IInAppNotificationService, InAppNotificationService>();
```

---

## WPF Button Subclassing

WinUI does not support subclassing `Button` as the XAML root element. Use `UserControl` wrapper instead:

```xml
<!-- WPF — Button subclass (works) -->
<Button x:Class="MyApp.Controls.CollapsibleButton">
    <!-- custom template -->
</Button>

<!-- Uno — DOES NOT WORK. Use UserControl wrapper instead -->
<UserControl x:Class="MyApp.Controls.CollapsibleButton">
    <Button x:Name="InnerButton">
        <!-- custom template -->
    </Button>
</UserControl>
```

---

## Canvas Overlay / Drawing Pattern

WPF apps with canvas-based overlays (e.g., drawing bounding boxes, selection rectangles) map to WinUI pointer events:

| WPF | WinUI / Uno |
|---|---|
| `MouseDown` handler | `PointerPressed` handler |
| `MouseMove` handler | `PointerMoved` handler |
| `MouseUp` handler | `PointerReleased` handler |
| `CaptureMouse()` | `CapturePointer(e.Pointer)` |
| `ReleaseMouseCapture()` | `ReleasePointerCapture(e.Pointer)` |
| `ZoomBorder` (custom) | `ZoomContentControl` (Uno Toolkit) |
| `ContextMenu` | `ContextFlyout` with `MenuFlyout` |
| `RoutedCommand` | Direct method call via interface |
| `ApplicationCommands.Undo` | Custom state stack (see below) |

### Canvas Undo/Redo with State Stack

WPF `RoutedCommand`-based undo has no equivalent. Use a simple state stack:

```csharp
private readonly Stack<List<WordBorderInfo>> _undoStack = new();

// Capture state BEFORE destructive operations (delete, merge, transform)
public void PushUndo()
{
    _undoStack.Push(
        _wordBorders.Select(wb => wb.ToInfo()).ToList()
    );
}

public void Undo()
{
    if (_undoStack.Count == 0) return;
    var state = _undoStack.Pop();
    RestoreWordBorders(state);
}

private void RestoreWordBorders(List<WordBorderInfo> state)
{
    _canvas.Children.Clear();
    foreach (var info in state)
    {
        var wb = new WordBorder(info) { Host = this }; // Host not serialized
        _canvas.Children.Add(wb);
    }
}
```

Key points:
- `WordBorder.ToInfo()` serializes position + word to a `WordBorderInfo` record
- `new WordBorder(info)` recreates from record
- Must set `wb.Host = this` after creation (host reference is not serialized)
- Each `MenuFlyoutItem` gets both a `Click` handler and a `KeyboardAccelerator` calling the same method

**ZoomContentControl notes:**
- Has no `AutoFit` property — omit it
- Use `MinZoomLevel` / `MaxZoomLevel` (not `MinZoomFactor` / `MaxZoomFactor`)
- `Canvas.SetLeft()` / `Canvas.SetTop()` work identically in WinUI

---

## Pages vs UserControls

WPF allows nesting Pages inside Pages. WinUI does not.

| Scenario | WPF | Uno Platform |
|---|---|---|
| Embedded content | Page nested in Page | `UserControl` nested in Page |
| Navigation target | Page in Frame | Page in Frame (same) |
| Reusable component | UserControl | UserControl (same) |
| Custom control | Control subclass | Control subclass or `UserControl` |

---

## Complete Runtime Gotchas Table

| Gotcha | Symptom | Solution |
|---|---|---|
| Wrong Material resource key | Page renders blank, no error | Cross-reference against safe key list in XAML reference |
| NavigationView with Visibility navigator | Content area blank | Use Frame with `uen:Region.Attached="true"` |
| Missing `IsDefault: true` on Shell route | App launches to blank screen | Add `IsDefault: true` to Shell RouteMap |
| `SoftwareBitmap` in service interface | Compile error on non-Windows | Use `Stream` or `byte[]` at service boundaries |
| Project namespace shadows `Uno.*` | "Type not found" or wrong resolution | Use `global::` prefix on all `Uno.*` types |
| `Application.Exit()` on Wasm | Not implemented exception | Acceptable; only used in shutdown scenarios |
| `CliWrap` / process APIs | Not available on non-Windows | Use `Condition` in csproj to exclude |
| XAML compiler stale cache | `XamlCompiler.exe` exit code 1 | `dotnet clean` or delete `obj/` folder |
| `{Binding StringFormat=...}` | Silent failure, no text rendered | Use `<Run>` elements or computed property |
| `Thread.Sleep()` on Wasm | UI freezes, deadlock | Use `await Task.Delay()` |
| `Task.Result` on Wasm | Deadlock (single-threaded) | Use `await` throughout |
| Pages nested inside Pages | XAML parse error | Use UserControl for embedded content |
| Button XAML root subclass | XAML parse error | Use UserControl wrapper |
| `ZoomContentControl.AutoFit` | Property not found | Omit; use `MinZoomLevel`/`MaxZoomLevel` |
| ToggleSwitch fires Toggled on init | Settings overwritten on page load | Use `_isLoading` guard pattern |

---

## Fullscreen Grab / Screen Capture Migration

WPF borderless transparent topmost windows become fullscreen Pages with P/Invoke capture:

| WPF | Uno Platform |
|---|---|
| `AllowsTransparency=True`, `WindowStyle=None` | `AppWindow.SetPresenter(AppWindowPresenterKind.FullScreen)` |
| `System.Drawing` screen capture | P/Invoke GDI `BitBlt` + SkiaSharp |
| Full-screen overlay with Canvas | Canvas with `PointerPressed/Moved/Released` |
| Direct screen access | `IScreenCaptureService` → `WindowsScreenCaptureService` (`#if WINDOWS`) |

### Capture Workflow (Windows)

1. Maximize window → Minimize (`ShowWindow(hwnd, SW_MINIMIZE=6)` via P/Invoke)
2. Capture screen (GDI BitBlt + SkiaSharp decode)
3. Restore maximized with screenshot as background
4. User draws selection region on dark translucent overlay
5. OCR the selected region
6. Auto-navigate back to EditText with result

### Window Management Helpers

```csharp
// Get AppWindow from HWND
var hwnd = WinRT.Interop.WindowNative.GetWindowHandle(App.MainWindow);
var windowId = Win32Interop.GetWindowIdFromWindow(hwnd);
var appWindow = AppWindow.GetFromWindowId(windowId);

// Enter fullscreen
appWindow.SetPresenter(AppWindowPresenterKind.FullScreen);

// Minimize for capture (P/Invoke)
[DllImport("user32.dll")] static extern bool ShowWindow(IntPtr hwnd, int nCmdShow);
ShowWindow(hwnd, 6); // SW_MINIMIZE

// Restore on exit (in Page.Unloaded)
appWindow.SetPresenter(AppWindowPresenterKind.Default);
```

### Canvas Overlay Styling

```xml
<!-- Dark translucent background -->
<Canvas Background="#60000000">
    <!-- Selection region: bright cutout -->
    <Border Background="#20FFFFFF" BorderBrush="White" BorderThickness="1" />
</Canvas>

<!-- Floating toolbar -->
<Border Background="#E0202020" CornerRadius="8" Padding="8">
    <Border.Shadow>
        <ThemeShadow />
    </Border.Shadow>
    <Border.Translation>
        <Vector3>0,0,32</Vector3>
    </Border.Translation>
    <!-- Mode buttons: Normal, Single Line, Table -->
</Border>
```

### Keyboard Shortcuts

Handle in `Page.KeyDown`:
- **Esc** = cancel and navigate back
- **S** = single-line mode
- **N** = normal mode
- **T** = table mode
- **E** = toggle edit-text-window

**Non-Windows fallback**: file picker or clipboard image (no screen capture API).

**OCR modes** after capture:
- Standard: multi-line text extraction
- Single Line: join all lines
- Table: grid-based extraction with column/row detection

---

## Global Hotkey Migration

WPF `HwndSource.AddHook` + Win32 `RegisterHotKey` → DI service with P/Invoke:

```csharp
public interface IHotKeyService : IDisposable
{
    void Register(int id, uint modifiers, uint vk);
    void UnregisterAll();
    event Action<int> HotKeyPressed;
}

#if WINDOWS
public class WindowsHotKeyService : IHotKeyService
{
    [DllImport("user32.dll")] static extern bool RegisterHotKey(IntPtr hwnd, int id, uint mod, uint vk);
    [DllImport("user32.dll")] static extern bool UnregisterHotKey(IntPtr hwnd, int id);

    // Handle WM_HOTKEY (0x0312) in WndProc
    // Use MOD_NOREPEAT to prevent auto-repeat spam
    // Hotkey IDs as constants: FullscreenGrabId=1, GrabFrameId=2, etc.
}
#endif
```

Key points:
- Requires a window handle (`HWND`) — obtain via `WinRT.Interop.WindowNative.GetWindowHandle()`
- `MOD_NOREPEAT` flag prevents held-key spam
- Service is `IDisposable` — call `UnregisterAll()` on app shutdown
- Non-Windows: hide hotkey settings UI entirely

---

## Web Search / URL Launch

```csharp
// WPF
Process.Start(new ProcessStartInfo(url) { UseShellExecute = true });

// Uno Platform (cross-platform)
await Windows.System.Launcher.LaunchUriAsync(new Uri(url));
// Desktop: opens browser. Wasm: opens new tab. Mobile: opens default browser.
// Also works for mailto: URIs
```

---

## Feature Parity Tracking

Don't mark phases "complete" until features are actually functional, not just scaffolded. Track per-area:

| Feature Area | How to Verify Parity |
|---|---|
| Menu items | Compare every `MenuFlyoutItem` in WPF vs Uno side-by-side |
| Toolbar buttons | Compare every `AppBarButton` / toolbar action |
| Settings toggles | Verify each toggle actually persists and applies |
| Keyboard shortcuts | Compare all `KeyboardAccelerator` entries against WPF `InputBindings` |

Quick wins first: menu items that just call existing utility methods (e.g., string transforms).

---

## Conditional Compilation in .csproj

For dependencies that only work on specific platforms:

```xml
<ItemGroup Condition="'$(TargetFramework)' == 'net9.0-windows10.0.22621'">
    <PackageReference Include="CliWrap" Version="3.6.0" />
</ItemGroup>

<ItemGroup Condition="'$(TargetFramework)' == 'net9.0-browserwasm'">
    <PackageReference Include="Uno.Wasm.Bootstrap" Version="9.0.0" />
</ItemGroup>
```

Use TFM conditions to include platform-specific NuGet packages rather than `#if` guards in code that references missing types.

---

## Cross-Platform Data Types at Service Boundaries

| Windows-Only Type | Cross-Platform Alternative | When to Convert |
|---|---|---|
| `SoftwareBitmap` | `Stream`, `byte[]`, `SKBitmap` | At service interface boundary |
| `Windows.Globalization.Language` | `CultureInfo` wrapped in custom record | At language service boundary |
| `Windows.Foundation.Rect` | `Windows.Foundation.Rect` | **No conversion needed** — provided by Uno.WinUI |
| `StorageFile` | `Stream` or file path `string` | At file service boundary |
| `BitmapImage` | `ImageSource` or `Stream` | At image service boundary |

---

## Migration Effort Estimation

Based on production migration data (37K LOC WPF → 13.2K LOC Uno, ~75% verified parity):

| Phase | Typical LOC | Effort |
|---|---|---|
| Scaffolding + Shell | ~500 | 1-2 days |
| First feature page + shared utilities | ~3,000 | 3-5 days |
| Service abstraction (per service) | ~500-1,000 | 1-2 days per service |
| Each additional page | ~800-2,000 | 1-3 days per page |
| Settings (with nested nav) | ~1,700 | 2-3 days |
| Platform polish (first-run, theme, notifications, capture, hotkeys) | ~2,200 | 3-5 days |
| Portable unit tests | ~1,200 | 1-2 days |
| Gap closure (missing commands, undo, parity) | ~500 | 1-2 days |

**Expect ~36% of original WPF LOC** in the Uno Platform port, covering ~75% feature parity. The reduction comes from MVUX eliminating boilerplate, DI replacing singletons, and Material theme replacing custom styling.

### What "75% Verified" Means

| Feature Area | Parity | What Works End-to-End | Remaining Gaps |
|---|---|---|---|
| EditText | **80%** | Text transforms, OCR paste, clipboard watcher, recent files, find/replace, regex, web search, QR code, font, always-on-top | AI menu, calc pane |
| GrabFrame | **60%** | Image load, OCR, undo/redo, table mode, move/resize, drag-drop, search | Translation, WPF code depth |
| QuickLookup | **65%** | CSV load, search, regex, copy, add row, insert-on-copy, send-to-ETW | Barcode, web sources, history |
| Settings | **80%** | All 6 pages persist, export/import, theme | StartupOnLogin registration |
| FullscreenGrab | **70%** | Fullscreen overlay, capture, OCR, modes, keyboard shortcuts | Single-click word, post-grab actions |
| System | **30%** | Hotkey service, background mode (minimize on close) | System tray, startup task |
| Dialogs | **90%** | FindReplace, RegexManager, BottomBar, PostGrab, settings export/import | — |

The remaining 25% is: AI features (new, not migration), translation (Windows AI API), barcode depth, calc pane (new UI), system tray (complex platform code). A user can edit text, OCR images, grab frames, look up items, and configure settings. The app builds and runs on Windows Desktop + WebAssembly with 248 passing unit tests.

---

## Common Mistakes

- Using `#if WINDOWS` in XAML files (not supported — use Visibility + code-behind)
- Passing Windows-only types across service interfaces
- Not guarding Windows-only features with `#if WINDOWS` (compile failure on other targets)
- Using route-based navigation for first-run (ContentDialog is simpler and more reliable)
- Nesting Pages inside Pages (use UserControl for embedded content)
- Subclassing Button as XAML root (use UserControl wrapper)
- Calling `Thread.Sleep()` or `Task.Result` on Wasm (causes deadlock)
- Not using TFM conditions in .csproj for platform-specific NuGet packages
- Marking migration phases "complete" when features are scaffolded but not functional
- Not doing element-by-element comparison (every MenuFlyoutItem, every toolbar button) for parity
