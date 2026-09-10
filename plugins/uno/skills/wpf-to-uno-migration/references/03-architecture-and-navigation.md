# Architecture and Navigation Reference

Shell setup, NavigationView region patterns, route registration, and the critical gotchas that cause blank pages during WPF-to-Uno migration.

---

## Shell Architecture

WPF apps with a main window containing a navigation sidebar map directly to Uno's Shell + NavigationView pattern.

### WPF Structure (Before)

```
MainWindow.xaml
├── NavigationView (sidebar)
│   ├── NavigationViewItem → PageA
│   ├── NavigationViewItem → PageB
│   └── NavigationViewItem → PageC
└── Frame (content area)
    └── Pages loaded via Frame.Navigate()
```

### Uno Structure (After)

```
ShellPage.xaml (wrapped in a ShellModel for MVUX)
├── NavigationView (uen:Region.Attached="true")
│   ├── NavigationViewItem (uen:Region.Name="PageA")
│   ├── NavigationViewItem (uen:Region.Name="PageB")
│   └── NavigationViewItem (uen:Region.Name="PageC")
└── Frame (uen:Region.Attached="true")
    └── Pages loaded via route registration
```

---

## NavigationView Region Pattern (CRITICAL)

This is the **#1 source of blank-page bugs** in WPF-to-Uno migrations.

### What Works

```xml
<NavigationView uen:Region.Attached="true">
    <NavigationView.MenuItems>
        <NavigationViewItem uen:Region.Name="EditText" Content="Edit Text">
            <NavigationViewItem.Icon>
                <FontIcon Glyph="&#xE70F;" />
            </NavigationViewItem.Icon>
        </NavigationViewItem>
        <NavigationViewItem uen:Region.Name="GrabFrame" Content="Grab Frame">
            <NavigationViewItem.Icon>
                <FontIcon Glyph="&#xE7C5;" />
            </NavigationViewItem.Icon>
        </NavigationViewItem>
    </NavigationView.MenuItems>

    <!-- MUST be a Frame with Region.Attached for NavigationView -->
    <Frame uen:Region.Attached="true" />
</NavigationView>
```

### What Does NOT Work

```xml
<!-- DO NOT USE — content renders blank inside NavigationView -->
<NavigationView>
    <Grid uen:Region.Attached="true" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="EditText" />
        <Grid uen:Region.Name="GrabFrame" />
    </Grid>
</NavigationView>
```

**Why:** NavigationView requires a Frame-based navigator. The Visibility navigator (which swaps content by toggling Visibility on child elements) only works with Panel/Grid containers **outside** of NavigationView, such as TabBar or standalone panel-based navigation.

### Rules

1. **NavigationView** → Always use `Frame` with `uen:Region.Attached="true"` as content
2. **TabBar / Panel** → Can use Visibility navigator with `uen:Region.Navigator="Visibility"`
3. Both the NavigationView AND the Frame need `uen:Region.Attached="true"`
4. NavigationViewItems use `uen:Region.Name` to map to route names

---

## Route Registration

Routes are registered in `App.xaml.cs` inside a `RegisterRoutes` method called during host configuration.

### Basic Pattern

```csharp
private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        new ViewMap<ShellPage, ShellModel>(),
        new ViewMap<EditTextPage, EditTextModel>(),
        new ViewMap<GrabFramePage, GrabFrameModel>(),
        new ViewMap<SettingsPage, SettingsModel>()
    );

    routes.Register(
        new RouteMap("", View: views.FindByViewModel<ShellModel>(),
            Nested: [
                new("Shell", View: views.FindByView<ShellPage>(), IsDefault: true,
                    Nested: [
                        new("EditText", View: views.FindByViewModel<EditTextModel>(), IsDefault: true),
                        new("GrabFrame", View: views.FindByViewModel<GrabFrameModel>()),
                        new("Settings", View: views.FindByViewModel<SettingsModel>()),
                    ]),
            ])
    );
}
```

### Critical Rules

- **`IsDefault: true` on the Shell route** is REQUIRED for automatic startup navigation. Without it, the app launches to a blank screen.
- **`IsDefault: true` on the first child route** (e.g., `EditText`) makes it the default page shown when the Shell loads.
- Route names in `RouteMap` must **exactly match** `uen:Region.Name` values in XAML.
- The root `RouteMap("")` with empty name is the entry point.

---

## Nested NavigationView (Settings Sub-Pages)

WPF apps with a SettingsWindow containing its own NavigationView (e.g., General, Display, Advanced sections) use the **exact same Frame pattern** when nested.

### Settings Page XAML

```xml
<Page x:Class="MyApp.Views.SettingsPage">
    <NavigationView uen:Region.Attached="true"
                    PaneDisplayMode="Left"
                    IsBackButtonVisible="Collapsed">
        <NavigationView.MenuItems>
            <NavigationViewItem uen:Region.Name="GeneralSettings" Content="General" />
            <NavigationViewItem uen:Region.Name="DisplaySettings" Content="Display" />
            <NavigationViewItem uen:Region.Name="AdvancedSettings" Content="Advanced" />
        </NavigationView.MenuItems>

        <!-- Same Frame pattern as the main Shell -->
        <Frame uen:Region.Attached="true" />
    </NavigationView>
</Page>
```

### Nested Route Registration

```csharp
new("Settings", View: views.FindByViewModel<SettingsModel>(),
    Nested: [
        new("GeneralSettings", View: views.FindByViewModel<GeneralSettingsModel>(), IsDefault: true),
        new("DisplaySettings", View: views.FindByViewModel<DisplaySettingsModel>()),
        new("AdvancedSettings", View: views.FindByViewModel<AdvancedSettingsModel>()),
    ]),
```

Each settings sub-page gets its own Model + Page + Route, just like top-level pages.

---

## WPF Dialog Windows → Uno Platform Patterns

### ContentDialog for Modal Dialogs

Every WPF child `Window` used as a modal dialog becomes a `ContentDialog`:

```csharp
// WPF
var dialog = new FindReplaceDialog();
dialog.Owner = this;
if (dialog.ShowDialog() == true)
{
    var result = dialog.SearchTerm;
}

// Uno Platform
var dialog = new FindReplaceDialog();
dialog.XamlRoot = this.XamlRoot;
var result = await dialog.ShowAsync();
if (result == ContentDialogResult.Primary)
{
    var searchTerm = dialog.SearchTerm;
}
```

### ContentDialog XAML Template

```xml
<ContentDialog
    x:Class="MyApp.Dialogs.FindReplaceDialog"
    Title="Find and Replace"
    PrimaryButtonText="Find"
    SecondaryButtonText="Replace"
    CloseButtonText="Cancel">

    <StackPanel Spacing="8">
        <TextBox x:Name="SearchBox" Header="Find" />
        <TextBox x:Name="ReplaceBox" Header="Replace with" />
        <CheckBox Content="Match case" />
    </StackPanel>
</ContentDialog>
```

### Advanced ContentDialog Patterns

- **Simple forms**: build UI programmatically in code-behind (`StackPanel` + `TextBox`es) — no dedicated XAML file needed
- **Complex dialogs**: create a dedicated XAML `ContentDialog` subclass in `Dialogs/` folder
- **Nested dialogs**: a ContentDialog can show another ContentDialog (e.g., RegexManager → delete confirmation)
- **Keep dialog open on button click**: `SecondaryButton` handler with `args.Cancel = true` — useful for "Reset Defaults" that shouldn't dismiss
- **`XamlRoot` is required**: dialog won't show without `dialog.XamlRoot = this.XamlRoot`

### When NOT to Use ContentDialog

- **First-run flow**: Use ContentDialog shown in `ShellPage.Loaded`, not route-based navigation (routes can't navigate back to Shell properly from root-level siblings)
- **Multi-step wizard**: Consider navigation to a dedicated page instead
- **Non-modal interaction**: Use `Flyout` or `TeachingTip` instead

---

## Two-Pane List Management Pattern

WPF apps with "Available Items" / "Enabled Items" list management (Add/Remove/MoveUp/MoveDown) port directly:

```csharp
// Both WPF and Uno use ObservableCollection<T>
public ObservableCollection<string> AvailableItems { get; } = new();
public ObservableCollection<string> EnabledItems { get; } = new();

public void AddItem()
{
    if (SelectedAvailable is { } item)
    {
        AvailableItems.Remove(item);
        EnabledItems.Add(item);
    }
}

public void MoveUp()
{
    var index = EnabledItems.IndexOf(SelectedEnabled);
    if (index > 0)
        EnabledItems.Move(index, index - 1);
}
```

This is one of the few patterns that migrates almost 1:1 from WPF.

---

## Common Mistakes

- Using Visibility navigator inside NavigationView (renders blank — must use Frame)
- Forgetting `IsDefault: true` on the Shell route (blank screen on launch)
- Forgetting `IsDefault: true` on the first child route (no default page selected)
- Route name mismatch between `RouteMap("EditText")` and `uen:Region.Name="editText"` (case-sensitive)
- Not adding `uen:Region.Attached="true"` to BOTH NavigationView AND Frame
- Trying to use route-based navigation for first-run flows (ContentDialog is simpler)
- Replicating WPF `Frame.Navigate()` calls instead of using declarative navigation
- Not adding `xmlns:uen="using:Uno.Extensions.Navigation.UI"` to XAML
