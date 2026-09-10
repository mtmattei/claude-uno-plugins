---
name: wpf-to-uno-migration
description: "Migrate WPF desktop applications to cross-platform Uno Platform apps. Phased workflow covering namespace/API mapping, XAML replacements, NavigationView region setup, Material Design theming, settings with IWritableOptions, service abstraction with DI, and platform-specific UI patterns. Use when: (1) Planning or executing a WPF-to-Uno migration, (2) Converting WPF Windows/dialogs to Uno Pages/ContentDialogs, (3) Replacing WPF settings/singletons with Uno Extensions DI, (4) Adapting WPF controls to WinUI/Material equivalents, (5) Debugging blank pages from bad resource keys or NavigationView misconfiguration. Do NOT use for: Silverlight migration (see silverlight-to-uno-migration), new Uno project setup (see uno-platform-agent), or general XAML optimization (see winui-xaml)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation and production migrations)"
metadata:
  version: "1.0.0"
  author: "Uno Platform"
  category: "uno-platform-migration"
  tags: [wpf, migration, cross-platform, uno-platform, winui]
---

# WPF to Uno Platform Migration

Phased workflow for migrating WPF desktop applications to cross-platform Uno Platform apps. Patterns are battle-tested from production migrations (37K LOC WPF app → 13.2K LOC Uno app at ~75% verified feature parity).

## Before You Start

### Prerequisites

1. **Install tooling**: Latest .NET SDK, Uno Platform templates (`dotnet new install Uno.Templates`), and `uno-check`
2. **Run `uno-check`** to validate your environment before writing any code
3. **Audit your WPF codebase** for migration scope:
   - Count usages of `x:Static`, `MultiBinding`, `Style.Triggers`, `DataTrigger` (all unsupported — need replacements)
   - Identify `DataGrid`, custom `Window` subclasses, WPF-UI controls (all need alternatives)
   - List all `System.Windows.*` namespace imports (all change to `Microsoft.UI.Xaml.*`)
   - Catalog platform-specific APIs: clipboard, file system, process, registry, COM (need `#if WINDOWS` guards)

### Key Architectural Decisions

| WPF Pattern | Uno Platform Replacement | Why |
|---|---|---|
| Multiple Windows | Single Window + Page navigation | Cross-platform apps use a single-window model |
| Static singletons | DI with `IServiceProvider` | Testable, cross-platform, MVUX-compatible |
| `Properties.Settings.Default` | `IWritableOptions<T>` | Reactive, type-safe, persisted automatically |
| Code-behind event handlers | MVUX Models with `IState<T>` / `IFeed<T>` | Reactive data flow, no manual UI updates |
| WPF MVVM (ICommand) | MVUX auto-generated commands | Public async methods become commands automatically |
| `RoutedCommand` / `CommandBindings` | Direct method calls + `KeyboardAccelerators` | WPF command routing doesn't exist in WinUI |
| `Process.Start(url)` | `Launcher.LaunchUriAsync(uri)` | Cross-platform (desktop, Wasm) |

## Migration Workflow

### Phase 1: Scaffold (~500 LOC)

1. **Create reference project** alongside your WPF solution:
   ```bash
   dotnet new unoapp -o MyApp.Uno --preset=recommended
   ```
2. **Verify it builds**: `dotnet build MyApp.Uno`
3. **Establish shared code project** (optional): portable models, interfaces, and utilities that both WPF and Uno can reference during incremental migration
4. **Set up GlobalUsings.cs** to reduce per-file import noise:
   ```csharp
   global using MyApp;
   global using MyApp.Interfaces;
   global using MyApp.Models;
   global using MyApp.Services;
   ```

### Phase 2: Shell and First Page (~3,000 LOC)

1. **Build the Shell** with NavigationView — this is the most critical architecture decision. See [references/03-architecture-and-navigation.md](references/03-architecture-and-navigation.md) for the exact pattern
2. **Port your simplest feature page first** (e.g., a text editor, settings viewer) to validate the full stack: Model → Page → Route → Navigation
3. **Port shared utilities**: string helpers, extension methods, converters — anything with no UI or platform dependency
4. **Port dialogs**: WPF child `Window` instances become `ContentDialog`

### Phase 3: Service Abstraction (~2,000 LOC)

1. **Define interfaces** for every platform-specific service (e.g., `IOcrEngine`, `IFileService`, `IClipboardService`)
2. **Register platform-specific implementations** with `#if WINDOWS` guards in `App.xaml.cs`:
   ```csharp
   #if WINDOWS
   services.AddSingleton<IOcrEngine, WindowsOcrEngine>();
   #endif
   ```
3. **Create service facades** that route to the correct implementation based on runtime context
4. **Replace all `Singleton<T>.Instance`** calls with constructor-injected interfaces

### Phase 4: Remaining Pages (~2,000-3,000 LOC per page)

Port pages in order of dependency, simplest first. For each page:
1. Create an MVUX Model with `IState<T>` for mutable data, `IFeed<T>` for read-only
2. Create the XAML Page referencing the Model
3. Register the route in `App.xaml.cs`
4. Build and test on at least two targets (Windows + one other)

### Phase 5: Settings and Theme (~1,700 LOC)

1. **Migrate settings**: `Properties.Settings.Default` → `IWritableOptions<AppSettings>` with `Section<AppSettings>()` in `UseConfiguration`. See [references/04-settings-theme-services.md](references/04-settings-theme-services.md)
2. **Migrate theme switching**: `App.SetTheme()` → `SystemThemeHelper.SetApplicationTheme()`
3. **Implement nested NavigationView** for settings sub-pages (same Frame pattern as main Shell)

### Phase 6: Platform Polish (~2,200 LOC)

1. **First-run flow**: Use `ContentDialog` shown in `ShellPage.Loaded`, not route-based navigation
2. **In-app notifications**: `InfoBar` overlay in ShellPage, replacing Windows toast notifications
3. **History/storage**: `ApplicationData.Current.LocalFolder` for cross-platform file storage
4. **Platform guards**: `#if WINDOWS` in code-behind for Windows-only features; `Visibility="Collapsed"` in XAML with code-behind toggle
5. **Screen capture**: Abstract behind `IScreenCaptureService`; Windows uses P/Invoke GDI BitBlt + SkiaSharp; other platforms fall back to file picker or clipboard
6. **Global hotkeys**: Abstract behind `IHotKeyService`; Windows uses P/Invoke `RegisterHotKey`/`WM_HOTKEY`; non-Windows hides the UI

### Phase 7: Tests and Gap Closure (~1,700 LOC)

1. **Port portable unit tests** first — string utilities, models, service logic (no UI dependency)
2. **Add MVUX model tests** for reactive state behavior
3. **Systematic gap closure**: compare WPF UI element-by-element (every `MenuFlyoutItem`, every toolbar button) against Uno to find missing commands
4. **Track parity percentage per feature area**, not just per phase — prevents "scaffolded but not functional" false completions

### Phase 8: Validate

1. Build all targets: `dotnet build` for each TFM
2. Test navigation: every route renders content (not blank)
3. Test settings: values persist across app restart
4. Test on non-Windows target to catch platform assumptions

## Critical Patterns

### NavigationView Region Navigation

This is the #1 source of blank-page bugs. **Must use Frame, not Grid/Panel with Visibility navigator.**

```xml
<!-- CORRECT: Frame inside NavigationView -->
<NavigationView uen:Region.Attached="true">
  <NavigationView.MenuItems>
    <NavigationViewItem uen:Region.Name="PageA" Content="Page A" />
    <NavigationViewItem uen:Region.Name="PageB" Content="Page B" />
  </NavigationView.MenuItems>
  <Frame uen:Region.Attached="true" />
</NavigationView>
```

```xml
<!-- WRONG: Visibility navigator inside NavigationView — renders blank -->
<Grid uen:Region.Attached="true" uen:Region.Navigator="Visibility">
  <Grid uen:Region.Name="PageA" />
</Grid>
```

Key rules:
- `IsDefault: true` on the Shell route is **required** for automatic startup navigation
- Nested NavigationView (e.g., Settings sub-pages) uses the **exact same Frame pattern**
- Visibility navigator only works **outside** NavigationView (e.g., TabBar, Panel)

### Material Theme Resource Keys

Wrong resource keys cause **silent runtime blank pages** — no compile error.

| Category | Correct (Uno Material) | Wrong (WPF/WinUI) |
|---|---|---|
| Typography | `TitleLarge`, `BodyMedium` | ~~`TitleLargeTextBlockStyle`~~ |
| Background | `SurfaceBrush`, `BackgroundBrush` | ~~`ApplicationPageBackgroundThemeBrush`~~ |
| Status | Use `ErrorBrush` or custom | ~~`SystemFillColorCautionBrush`~~ |

Pattern: `{Category}{Size}` — `DisplayLarge`, `HeadlineMedium`, `TitleSmall`, `BodyMedium`, `LabelSmall`

### Namespace Collision with `global::`

If your project namespace starts with a common prefix (e.g., `MyApp.Uno`), it can shadow `Uno.*` namespaces:

```csharp
// ERROR: Resolves to MyApp.Uno.Extensions.Configuration instead of Uno.Extensions.Configuration
IWritableOptions<AppSettings> settings;

// FIX: Use global:: prefix
global::Uno.Extensions.Configuration.IWritableOptions<AppSettings> settings;
global::Uno.Toolkit.UI.SystemThemeHelper.SetApplicationTheme(xamlRoot, theme);
```

### WPF Window to ContentDialog

Every WPF child window becomes a `ContentDialog`:

| WPF | Uno Platform |
|---|---|
| `new MyDialog().ShowDialog()` | `new MyDialog().ShowAsync()` |
| `DialogResult = true` | `ContentDialogResult.Primary` |
| `Owner = this` | `XamlRoot = this.XamlRoot` |

Advanced patterns:
- **Simple forms**: build UI programmatically in code-behind (`StackPanel` + `TextBox`es)
- **Complex dialogs**: dedicated XAML `ContentDialog` subclass in `Dialogs/` folder
- **Nested dialogs**: a ContentDialog can show another ContentDialog (e.g., manager → delete confirmation)
- **Keep dialog open**: `SecondaryButton` handler with `args.Cancel = true` (useful for "Reset Defaults")

### WPF RoutedCommand → KeyboardAccelerators

WPF's `RoutedCommand` + `CommandBindings` system has no equivalent in WinUI:

| WPF | Uno Platform |
|---|---|
| `RoutedCommand` + `CommandBinding` | Direct `Click` handler calling the same method |
| `ApplicationCommands.Undo/Redo` | Custom undo stack (see canvas undo pattern in ref/05) |
| `InputGestureText` (display only) | `KeyboardAccelerators` (functional — actually fires the command) |

Pattern: each `MenuFlyoutItem` gets both a `Click` handler and a `KeyboardAccelerator` that call the same method.

### Settings Migration Pattern

```csharp
// WPF: Properties.Settings.Default.MyValue = x;
// Uno: IWritableOptions<AppSettings> with MVUX

public partial record SettingsModel(IWritableOptions<AppSettings> Settings)
{
    public IState<bool> IsFeatureEnabled => State.Async(async ct =>
    {
        var s = await Settings.GetAsync(ct);
        return s.IsFeatureEnabled;
    });

    public async ValueTask ToggleFeature(bool value) =>
        await Settings.UpdateAsync(s => s with { IsFeatureEnabled = value });
}
```

`Section<AppSettings>()` in `UseConfiguration` registers **both** `IOptions<AppSettings>` and `IWritableOptions<AppSettings>` — no extra registration needed.

## Common Mistakes

**Architecture:**
- Replicating WPF's multi-window model instead of adopting single-window + Page navigation
- Keeping static singletons instead of migrating to DI (breaks MVUX, testability, and cross-platform)
- Porting WPF code-behind event handlers verbatim instead of using MVUX reactive patterns
- Using route-based navigation for first-run flows (ContentDialog is simpler and avoids back-stack issues)
- Marking migration phases "complete" when features are scaffolded but not functional — track parity per feature area

**XAML:**
- Using long WinUI-style typography keys (`TitleLargeTextBlockStyle`) instead of Material short keys (`TitleLarge`)
- Referencing WPF/WinUI system brushes that don't exist in Material theme (causes silent blank pages)
- Using `{Binding StringFormat=...}` which is not supported — use `<Run>` elements or computed properties
- Setting `Page.Background` explicitly instead of inheriting from parent theme

**Navigation:**
- Using Visibility navigator inside NavigationView (renders blank — must use Frame)
- Forgetting `IsDefault: true` on the Shell route (app launches to blank screen)
- Not adding `uen:Region.Attached="true"` to both NavigationView AND its child Frame

**Platform:**
- Using `#if WINDOWS` in XAML (not supported) — use Visibility + code-behind toggle instead
- Passing `SoftwareBitmap` across service boundaries (not cross-platform) — use `Stream` or `byte[]`
- Subclassing `Button` as XAML root element (not supported in WinUI) — use `UserControl` wrapper
- Forgetting `global::` prefix when project namespace shadows `Uno.*`

**Build:**
- Stale XAML compiler cache causing `XamlCompiler.exe` exit code 1 — run `dotnet clean` or delete `obj/`
- Not running `uno-check` before starting migration
- Not building all TFMs regularly during migration (platform bugs compound)

## Related Skills

| Skill | Use instead when... |
|---|---|
| `silverlight-to-uno-migration` | Migrating from Silverlight |
| `uno-build-troubleshoot` | Upgrading .NET versions or fixing build errors (UNOB0011, UNOB0013) |
| `uno-platform-agent` | Creating a new Uno project from scratch (not migrating an existing WPF app) |
| `uno-navigation` | Setting up navigation or NavigationView in a new project (this skill covers migration-specific navigation patterns) |
| `uno-extensions-services` | Configuring DI, auth, HTTP, or logging (not migration-specific) |
| `uno-material` | Installing Material theme from scratch (this skill covers migrating WPF theme patterns) |
| `winui-xaml` | XAML layout, binding, and performance best practices (not migration-specific) |
| `uno-toolkit` | Using Toolkit controls (AutoLayout, SafeArea, TabBar) after migration |

## Detailed References

Read the reference file matching your current migration phase:

- [references/01-namespace-api-mapping.md](references/01-namespace-api-mapping.md) — Read when doing the initial namespace sweep and API mapping pass
- [references/02-xaml-patterns-and-controls.md](references/02-xaml-patterns-and-controls.md) — Read when replacing unsupported XAML features, mapping WPF controls, or debugging blank pages from bad resource keys
- [references/03-architecture-and-navigation.md](references/03-architecture-and-navigation.md) — Read when setting up Shell, NavigationView, route registration, or nested navigation
- [references/04-settings-theme-services.md](references/04-settings-theme-services.md) — Read when migrating settings, theme switching, or abstracting platform-specific services with DI
- [references/05-platform-specific-gotchas.md](references/05-platform-specific-gotchas.md) — Read when handling Windows-only features, ContentDialog conversions, first-run flows, screen capture, global hotkeys, undo/redo, or runtime gotchas
