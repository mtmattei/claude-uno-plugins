# Namespace and API Mapping Reference

Complete reference for the WPF-to-Uno namespace migration and API mapping pass. Use this as a checklist during the initial codebase sweep.

---

## Namespace Find-and-Replace Sequence

**Order matters.** Replace specific sub-namespaces before the root to avoid double-replacement bugs.

```
1. System.Windows.Threading     →  Microsoft.UI.Dispatching
2. System.Windows.Controls      →  Microsoft.UI.Xaml.Controls
3. System.Windows.Media          →  Microsoft.UI.Xaml.Media
4. System.Windows.Input          →  Microsoft.UI.Xaml.Input
5. System.Windows.Data           →  Microsoft.UI.Xaml.Data
6. System.Windows.Shapes         →  Microsoft.UI.Xaml.Shapes
7. System.Windows.Documents      →  Microsoft.UI.Xaml.Documents
8. System.Windows.Navigation     →  Microsoft.UI.Xaml.Navigation
9. System.Windows.Automation     →  Microsoft.UI.Xaml.Automation
10. System.Windows                →  Microsoft.UI.Xaml          (do this LAST)
```

**Common Mistakes:**
- Replacing `System.Windows` before sub-namespaces, producing `Microsoft.UI.Xaml.Xaml.Controls`
- Missing the `Threading` → `Dispatching` change (it breaks the pattern)
- Not updating `xmlns:local="clr-namespace:..."` to `xmlns:local="using:..."` in XAML files
- Forgetting assembly-qualified type names in string-based configs (converters, custom types)

**Note:** The default XAML namespace URI (`http://schemas.microsoft.com/winfx/2006/xaml/presentation`) is shared between WPF and Uno Platform — do not change `xmlns=` declarations.

---

## Core API Mapping Table

| WPF Type/Pattern | Uno Platform Equivalent | Notes |
|---|---|---|
| `Window` | `Page` | Uno uses single-window model; each "window" becomes a Page |
| `Window.ShowDialog()` | `ContentDialog.ShowAsync()` | Modal dialogs are ContentDialog instances |
| `NavigationWindow` | `NavigationView` + Frame + Region | Use Navigation Extensions, not Frame.Navigate |
| `{Binding Path=X}` | `{x:Bind X}` | `x:Bind` preferred for type safety and perf; `{Binding}` still works |
| `{Binding StringFormat=...}` | Multiple `<Run>` elements or computed property | `StringFormat` on `{Binding}` is **not supported** |
| `MultiBinding` | Multiple `<Run>` or computed property | Not supported; see XAML patterns reference |
| `x:Static` | `{StaticResource}` | Not supported; define resources in App.xaml or ResourceDictionary |
| `Style.Triggers` | `VisualStateManager` | Property triggers do not exist in WinUI |
| `DataTrigger` | `StateTrigger` in `VisualStateManager` | Declarative state via `StateTrigger.IsActive` binding |
| `EventTrigger` | Command binding or code-behind event | No trigger system; use MVVM commands |
| `ICommand` (custom impl) | `[RelayCommand]` or MVUX auto-commands | Source generators eliminate boilerplate |
| `RoutedUICommand` | Button with `Command` binding | WPF command routing model does not exist |
| `DataGrid` | `ListView` with custom `ItemTemplate` | No built-in DataGrid; use ListView with GridView |
| `Dispatcher.Invoke()` | `DispatcherQueue.TryEnqueue()` | Async by default; no synchronous Invoke equivalent |
| `Dispatcher.BeginInvoke()` | `DispatcherQueue.TryEnqueue()` | Same replacement for both sync and async dispatch |
| `MessageBox.Show()` | `ContentDialog.ShowAsync()` | Set `XamlRoot = this.XamlRoot` on the dialog |
| `Clipboard.SetText()` | `DataPackage` + `Clipboard.SetContent()` | WinUI clipboard uses DataPackage wrapper |
| `OpenFileDialog` | `FileOpenPicker` | Use `WinRT.Interop` for HWND on Windows |
| `SaveFileDialog` | `FileSavePicker` | Use `WinRT.Interop` for HWND on Windows |
| `UserControl` | `UserControl` | Same concept, different namespace |
| `ContentControl` | `ContentControl` | Same concept, different namespace |
| `ContextMenu` | `ContextFlyout` with `MenuFlyout` | Flyout system replaces ContextMenu |
| `ToolTip` | `ToolTipService.ToolTip` | Attached property pattern |
| `RoutedCommand` + `CommandBindings` | Direct `Click` handler + `KeyboardAccelerators` | WPF command routing doesn't exist |
| `ApplicationCommands.Undo/Redo` | Custom undo stack (`Stack<T>`) | WinUI TextBox has Ctrl+Z but no programmatic API |
| `InputGestureText` (display only) | `KeyboardAccelerators` (functional) | Actually fires the command, not just display |
| `Process.Start(url, UseShellExecute)` | `Launcher.LaunchUriAsync(new Uri(url))` | Cross-platform: desktop opens browser, Wasm opens tab |

---

## Input and Pointer Events

| WPF Event | WinUI / Uno Event | Notes |
|---|---|---|
| `MouseDown` | `PointerPressed` | Pointer events unify mouse, touch, pen |
| `MouseMove` | `PointerMoved` | |
| `MouseUp` | `PointerReleased` | |
| `MouseEnter` | `PointerEntered` | |
| `MouseLeave` | `PointerExited` | |
| `CaptureMouse()` | `CapturePointer(e.Pointer)` | Pass the specific pointer |
| `ReleaseMouseCapture()` | `ReleasePointerCapture(e.Pointer)` | |
| `Mouse.GetPosition()` | `e.GetCurrentPoint(element).Position` | Position relative to element |
| `KeyDown` (with `Key` enum) | `KeyDown` (with `VirtualKey` enum) | Enum values differ |

---

## Layout and Rendering

| WPF | Uno Platform | Notes |
|---|---|---|
| `StackPanel` | `StackPanel` or `AutoLayout` | AutoLayout (Uno Toolkit) preferred for Figma-like spacing |
| `DockPanel` | `Grid` with row/column definitions | No DockPanel equivalent; use Grid |
| `WrapPanel` | `ItemsWrapGrid` or AutoLayout with wrap | Limited support |
| `Canvas.SetLeft/SetTop` | `Canvas.SetLeft/SetTop` | Identical API |
| `ZoomBorder` (custom) | `ZoomContentControl` (Uno Toolkit) | Use `MinZoomLevel`/`MaxZoomLevel`, not `MinZoomFactor` |
| `RenderTransform` | `RenderTransform` | Same API, slightly different behavior on some platforms |
| `Effect` (blur, shadow) | `ThemeShadow` + `Translation` | WPF effects have no direct equivalent; use elevation |
| `SymbolIcon` (WPF-UI) | `FontIcon` with Segoe Fluent Icons glyphs | Different icon system |

---

## Data and Storage

| WPF | Uno Platform | Notes |
|---|---|---|
| `Properties.Settings.Default` | `IWritableOptions<T>` | See settings reference for full pattern |
| `Singleton<T>.Instance` | DI-registered `ISomethingService` | Constructor injection via MVUX Model |
| Direct database access | HTTP client (Kiota/Refit) | Direct DB not available on Wasm/mobile |
| `System.IO.File.Read/Write` | `ApplicationData.Current.LocalFolder` | Sandboxed storage; async on all platforms |
| `IsolatedStorage` | `ApplicationData.Current.LocalFolder` | Same replacement for both WPF and Silverlight |
| WCF services | gRPC or REST with Kiota/Refit | WCF not supported on non-Windows |
| `Process.Start()` | `#if WINDOWS` guard | Only available on Windows target |

---

## Threading and Async

| WPF | Uno Platform | Notes |
|---|---|---|
| `Dispatcher.Invoke(() => ...)` | `DispatcherQueue.TryEnqueue(() => ...)` | No synchronous dispatch; always async |
| `BackgroundWorker` | `Task.Run()` + async/await | Modern async pattern |
| `Task.Result` (blocking) | `await` | Never block on Wasm — causes deadlock |
| `Thread.Sleep()` | `await Task.Delay()` | Wasm is single-threaded; Sleep blocks the UI |

---

## Validation Checklist

After the namespace sweep, verify:

- [ ] No remaining `System.Windows.*` imports in any `.cs` file
- [ ] No `clr-namespace:` in any XAML file (should be `using:`)
- [ ] No `x:Static` usage anywhere
- [ ] No `MultiBinding` usage anywhere
- [ ] No `Style.Triggers`, `DataTrigger`, or `EventTrigger` usage
- [ ] No `{Binding StringFormat=...}` usage
- [ ] No `Dispatcher.Invoke()` or `Dispatcher.BeginInvoke()` calls
- [ ] No `MessageBox.Show()` calls
- [ ] No `Thread.Sleep()` calls (especially in async code)
- [ ] All `RoutedUICommand` instances replaced with MVVM commands
- [ ] `dotnet build` succeeds for at least one TFM
