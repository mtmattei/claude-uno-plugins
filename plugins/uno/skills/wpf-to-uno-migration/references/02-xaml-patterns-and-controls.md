# XAML Patterns, Controls, and Material Theme Reference

Detailed patterns for replacing unsupported WPF XAML features, mapping WPF controls to WinUI/Uno equivalents, and using Material Design resource keys correctly.

---

## Unsupported XAML Feature Replacements

### x:Static → StaticResource

```xml
<!-- WPF — NOT SUPPORTED in Uno -->
<TextBlock Text="{x:Static local:Constants.AppTitle}" />

<!-- Uno Platform — define in App.xaml or ResourceDictionary -->
<x:String x:Key="AppTitle">My Application</x:String>

<!-- Reference it -->
<TextBlock Text="{StaticResource AppTitle}" />
```

**Why it fails:** The Uno Platform XAML parser does not support `x:Static`. Do not attempt to create a custom markup extension to emulate it — custom markup extensions are not supported either.

### MultiBinding → Run Elements or Computed Property

```xml
<!-- WPF — NOT SUPPORTED in Uno -->
<TextBlock>
    <TextBlock.Text>
        <MultiBinding StringFormat="{}{0} - {1}">
            <Binding Path="FirstName" />
            <Binding Path="LastName" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>

<!-- Uno Platform Option 1: Multiple Run elements -->
<TextBlock>
    <Run Text="{x:Bind ViewModel.FirstName}" />
    <Run Text=" - " />
    <Run Text="{x:Bind ViewModel.LastName}" />
</TextBlock>

<!-- Uno Platform Option 2: Computed property in ViewModel -->
<!-- public string FullName => $"{FirstName} - {LastName}"; -->
<TextBlock Text="{x:Bind ViewModel.FullName}" />
```

**Note:** `{Binding StringFormat=...}` is ALSO not supported. This catches WPF developers who try it as a MultiBinding workaround.

### Style.Triggers / DataTrigger → VisualStateManager

```xml
<!-- WPF — NOT SUPPORTED in Uno -->
<Style.Triggers>
    <DataTrigger Binding="{Binding IsActive}" Value="True">
        <Setter Property="Background" Value="Green" />
    </DataTrigger>
</Style.Triggers>

<!-- Uno Platform — VisualStateManager with StateTrigger -->
<VisualStateManager.VisualStateGroups>
    <VisualStateGroup>
        <VisualState x:Name="Active">
            <VisualState.StateTriggers>
                <StateTrigger IsActive="{x:Bind ViewModel.IsActive, Mode=OneWay}" />
            </VisualState.StateTriggers>
            <VisualState.Setters>
                <Setter Target="RootBorder.Background" Value="Green" />
            </VisualState.Setters>
        </VisualState>
    </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

**Why VisualStateManager:** WinUI intentionally dropped WPF triggers for performance and consistency. `StateTrigger` and `AdaptiveTrigger` provide declarative state management without the overhead of WPF's trigger evaluation system.

### EventTrigger → Command Binding

```xml
<!-- WPF — NOT SUPPORTED in Uno -->
<Button>
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="Click">
            <i:InvokeCommandAction Command="{Binding SaveCommand}" />
        </i:EventTrigger>
    </i:Interaction.Triggers>
</Button>

<!-- Uno Platform — direct Command binding -->
<Button Command="{x:Bind ViewModel.SaveCommand}" Content="Save" />

<!-- Or with MVUX auto-command (method becomes command automatically) -->
<!-- public async ValueTask Save() { ... } -->
```

---

## WPF Control → WinUI/Uno Control Mapping

| WPF / WPF-UI Control | WinUI / Uno Equivalent | Migration Notes |
|---|---|---|
| `FluentWindow` (WPF-UI) | Standard `Window` | Uno doesn't need a FluentWindow wrapper |
| `DataGrid` | `ListView` with `ItemTemplate` | No built-in DataGrid; design with ListView + Grid columns in template |
| `Standalone Window` (child) | `ContentDialog` | See ContentDialog pattern in platform reference |
| `MessageBox` | `ContentDialog` | `ShowAsync()` with Primary/Secondary/Close buttons |
| `ContextMenu` | `ContextFlyout` with `MenuFlyout` | Attach to any FrameworkElement |
| `ToolBar` | `CommandBar` | Different API but same visual concept |
| `StatusBar` | Custom `Grid` at page bottom | No built-in StatusBar |
| `Expander` (WPF-UI) | `Expander` (WinUI built-in) | Identical API in WinUI |
| `ToggleSwitch` | `ToggleSwitch` | Identical API |
| `HyperlinkButton` | `HyperlinkButton` | Identical API |
| `SymbolIcon` (WPF-UI) | `FontIcon` with glyph | Use Segoe Fluent Icons or Material Symbols |
| `NumberBox` | `NumberBox` | Identical API |
| `RatingControl` | `RatingControl` | Identical API |
| `TreeView` | `TreeView` | Identical API |
| `TabControl` | `TabView` or `TabBar` (Toolkit) | TabBar preferred for bottom navigation |
| `RichTextBox` | `RichEditBox` | Different API; not all features supported cross-platform |
| `WrapPanel` | `ItemsWrapGrid` | Limited; consider AutoLayout |
| `DockPanel` | `Grid` | No DockPanel; use Grid rows/columns |

---

## Material Design Resource Keys (CRITICAL)

### The Blank Page Problem

Using a resource key that doesn't exist in Uno Material causes a **silent runtime failure** — the page renders blank with no compile error and no runtime exception. This is the single most common debugging trap in WPF-to-Uno migration.

### Typography Styles

Uno Material uses **short keys** following the pattern `{Category}{Size}`:

| Material Style Key | Usage |
|---|---|
| `DisplayLarge`, `DisplayMedium`, `DisplaySmall` | Hero text, large headings |
| `HeadlineLarge`, `HeadlineMedium`, `HeadlineSmall` | Page headings |
| `TitleLarge`, `TitleMedium`, `TitleSmall` | Section titles, app bars |
| `BodyLarge`, `BodyMedium`, `BodySmall` | Body text, descriptions |
| `LabelLarge`, `LabelMedium`, `LabelSmall` | Buttons, captions, metadata |

**WRONG keys (will cause blank pages):**
- ~~`TitleLargeTextBlockStyle`~~ (WinUI-style long name)
- ~~`BodyTextBlockStyle`~~ (WinUI generic name)
- ~~`CaptionTextBlockStyle`~~ (WinUI name)
- ~~`SubtitleTextBlockStyle`~~ (WinUI name)

```xml
<!-- WRONG — causes blank page -->
<TextBlock Style="{StaticResource TitleLargeTextBlockStyle}" />

<!-- CORRECT -->
<TextBlock Style="{StaticResource TitleLarge}" />
```

### Brush Resources

Auto-generated from `ColorPaletteOverride.xaml`:

| Safe Brush Name | Usage |
|---|---|
| `BackgroundBrush`, `OnBackgroundBrush` | Page/surface backgrounds and text on them |
| `SurfaceBrush`, `OnSurfaceBrush` | Card/container surfaces |
| `SurfaceVariantBrush`, `OnSurfaceVariantBrush` | Secondary surfaces |
| `PrimaryBrush`, `OnPrimaryBrush` | Primary action elements |
| `SecondaryBrush`, `OnSecondaryBrush` | Secondary action elements |
| `TertiaryBrush`, `OnTertiaryBrush` | Tertiary accents |
| `ErrorBrush`, `OnErrorBrush` | Error states |
| `OutlineBrush` | Borders and dividers |

**WRONG brush names (do not exist in Material):**
- ~~`ApplicationPageBackgroundThemeBrush`~~ (WinUI system resource)
- ~~`CardBackgroundFillColorDefaultBrush`~~ (WinUI system resource)
- ~~`SystemFillColorCautionBrush`~~ (WinUI system resource)
- ~~`SystemFillColorSuccessBrush`~~ (WinUI system resource)

**Best practice:** Prefer omitting `Page.Background` entirely to inherit from the parent theme, rather than setting it explicitly.

### Verifying Resource Keys

If a page renders blank:

1. Check every `{StaticResource X}` and `{ThemeResource X}` in the XAML
2. Cross-reference against the safe lists above
3. Remove suspicious resources one at a time until the page renders
4. Replace with known-safe Material keys

---

## XAML Namespace Declarations

```xml
<!-- Standard Uno Platform page namespaces -->
<Page
    x:Class="MyApp.Views.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp.Views"
    xmlns:uen="using:Uno.Extensions.Navigation.UI"
    xmlns:utu="using:Uno.Toolkit.UI"
    xmlns:um="using:Uno.Material">
```

**Key changes from WPF:**
- `clr-namespace:` → `using:` (WinUI XAML syntax)
- Add `uen:` for Navigation Extensions
- Add `utu:` for Toolkit controls (SafeArea, AutoLayout, etc.)
- Add `um:` for Material-specific controls

---

## Common Mistakes

- Using WinUI-style long typography keys instead of Material short keys (blank page, no error)
- Referencing WPF/WinUI system brushes in Material-themed apps (blank page, no error)
- Using `{Binding StringFormat=...}` as a MultiBinding workaround (also not supported)
- Using `clr-namespace:` instead of `using:` in xmlns declarations
- Attempting custom markup extensions to emulate `x:Static` (not supported)
- Not searching resource dictionaries and styles for `MultiBinding` (only checking page XAML)
- Setting explicit `Page.Background` that conflicts with Material theme inheritance
