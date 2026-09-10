# Styling and Theming

## Impact: CRITICAL
Lightweight styling, ResourceExtensions, StatusBarExtensions, VisualStateManagerExtensions, and SystemThemeHelper control the visual appearance and theme behavior of Uno Toolkit apps. Misplacing theme dictionaries, using wrong key patterns, attaching VisualStates to the wrong element, or using `x:Key="Dark"` instead of `"Default"` causes silent styling failures.

---

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

## Toolkit Lightweight Styling

**Impact: CRITICAL**

Override specific resource keys consumed by Toolkit control templates to customize appearance without redefining templates. This keeps customizations upgrade-safe.

### Resource Key Pattern

`{ControlName}{State}{Property}`

| Segment | Examples |
|---------|----------|
| ControlName | `NavigationBar`, `TabBar`, `Card`, `Chip`, `Divider`, `DrawerControl` |
| State | *(empty)* = rest, `PointerOver`, `Pressed`, `Disabled`, `Focused`, `Selected` |
| Property | `Background`, `Foreground`, `BorderBrush`, `FontSize`, `CornerRadius` |

### CORRECT
```xml
<SolidColorBrush x:Key="NavigationBarBackground" Color="#1A1A2E" />
<SolidColorBrush x:Key="TabBarItemForegroundSelected" Color="{ThemeResource PrimaryColor}" />
<SolidColorBrush x:Key="ChipBackgroundPointerOver" Color="#E8E8E8" />
```

### WRONG
```xml
<!-- WRONG: Inventing state names -->
<SolidColorBrush x:Key="NavigationBarBackgroundHover" Color="Red" />
<!-- FIX: Use "PointerOver" not "Hover" -->

<!-- WRONG: Wrong segment order (State before Property) -->
<SolidColorBrush x:Key="TabBarItemSelectedForeground" Color="Blue" />
<!-- FIX: TabBarItemForegroundSelected (Property then State) -->
```

---

## Three Override Scopes

**Impact: CRITICAL**

### 1. App Level (App.xaml) -- Global

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <MaterialToolkitTheme />
        </ResourceDictionary.MergedDictionaries>

        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="NavigationBarBackground" Color="#FFFFFF" />
            </ResourceDictionary>
            <ResourceDictionary x:Key="Default">
                <!-- "Default" = Dark theme -->
                <SolidColorBrush x:Key="NavigationBarBackground" Color="#1A1A2E" />
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

### 2. Page Level (Page.Resources) -- Scoped to one page

```xml
<Page.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="TabBarBackground" Color="#F5F5F5" />
            </ResourceDictionary>
            <ResourceDictionary x:Key="Default">
                <SolidColorBrush x:Key="TabBarBackground" Color="#2D2D44" />
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Page.Resources>
```

### 3. Control Level (Control.Resources) -- Single control

```xml
<utu:NavigationBar Content="Special Page">
    <utu:NavigationBar.Resources>
        <ResourceDictionary>
            <ResourceDictionary.ThemeDictionaries>
                <ResourceDictionary x:Key="Light">
                    <SolidColorBrush x:Key="NavigationBarBackground" Color="OrangeRed" />
                </ResourceDictionary>
                <ResourceDictionary x:Key="Default">
                    <SolidColorBrush x:Key="NavigationBarBackground" Color="DarkOrange" />
                </ResourceDictionary>
            </ResourceDictionary.ThemeDictionaries>
        </ResourceDictionary>
    </utu:NavigationBar.Resources>
</utu:NavigationBar>
```

---

## ThemeDictionaries Keys

**Impact: CRITICAL**

| x:Key | Theme |
|-------|-------|
| `"Light"` | Light theme |
| `"Default"` | Dark theme |

### CORRECT
```xml
<ResourceDictionary.ThemeDictionaries>
    <ResourceDictionary x:Key="Light">
        <!-- Light theme overrides -->
    </ResourceDictionary>
    <ResourceDictionary x:Key="Default">
        <!-- Dark theme overrides -->
    </ResourceDictionary>
</ResourceDictionary.ThemeDictionaries>
```

### WRONG
```xml
<!-- WRONG: "Dark" is not a valid key — these overrides are silently ignored -->
<ResourceDictionary x:Key="Dark">
    <SolidColorBrush x:Key="NavigationBarBackground" Color="#1A1A2E" />
</ResourceDictionary>
<!-- FIX: Use x:Key="Default" for dark theme -->

<!-- WRONG: Overriding outside ThemeDictionaries — won't respond to theme changes -->
<SolidColorBrush x:Key="NavigationBarBackground" Color="Red" />
<!-- FIX: Wrap in ThemeDictionaries with Light and Default keys -->
```

---

## ResourceExtensions.Resources

**Impact: HIGH**

Attach a `ResourceDictionary` directly to a `Style` to create reusable lightweight-styling variants without polluting page-level resources. This is a Uno Toolkit extension (`utu:ResourceExtensions.Resources`).

### CORRECT
```xml
xmlns:utu="using:Uno.Toolkit.UI"

<!-- Reusable danger-styled NavigationBar variant -->
<Style x:Key="DangerNavigationBarStyle"
       TargetType="utu:NavigationBar"
       BasedOn="{StaticResource NavigationBarStyle}">
    <Setter Property="utu:ResourceExtensions.Resources">
        <Setter.Value>
            <ResourceDictionary>
                <ResourceDictionary.ThemeDictionaries>
                    <ResourceDictionary x:Key="Light">
                        <SolidColorBrush x:Key="NavigationBarBackground" Color="#DC2626" />
                    </ResourceDictionary>
                    <ResourceDictionary x:Key="Default">
                        <SolidColorBrush x:Key="NavigationBarBackground" Color="#EF4444" />
                    </ResourceDictionary>
                </ResourceDictionary.ThemeDictionaries>
            </ResourceDictionary>
        </Setter.Value>
    </Setter>
</Style>

<!-- Usage -->
<utu:NavigationBar Content="Delete Confirmation"
                   Style="{StaticResource DangerNavigationBarStyle}" />
```

### WRONG
```xml
<!-- WRONG: Applying ResourceExtensions.Resources directly on a control instance -->
<!-- Use Control.Resources for per-instance overrides instead -->
<utu:NavigationBar Content="Title"
                   utu:ResourceExtensions.Resources="{StaticResource SomeDictionary}" />
<!-- FIX: ResourceExtensions.Resources is designed for Style setters, not direct control use -->
```

---

## StatusBarExtensions (iOS/Android Only)

**Impact: MEDIUM**

Control mobile status bar foreground and background from XAML via attached properties on `Page`.

### Foreground Values

| Value | Effect |
|-------|--------|
| `Dark` | Dark text/icons (for light backgrounds) |
| `Light` | Light text/icons (for dark backgrounds) |
| `Auto` | Follows current app theme (light fg in dark mode) |
| `AutoInverse` | Opposite of current theme |

### CORRECT
```xml
<!-- Light page with dark status bar text -->
<Page utu:StatusBar.Foreground="Dark"
      utu:StatusBar.Background="{ThemeResource SurfaceBrush}">
    <!-- Page content -->
</Page>

<!-- Dark header page with light status bar text -->
<Page utu:StatusBar.Foreground="Light"
      utu:StatusBar.Background="{ThemeResource PrimaryBrush}">
    <!-- Page content -->
</Page>

<!-- Theme-aware (recommended default) -->
<Page utu:StatusBar.Foreground="Auto"
      utu:StatusBar.Background="{ThemeResource SurfaceBrush}">
    <!-- Page content -->
</Page>
```

### WRONG
```xml
<!-- WRONG: Applying StatusBar extensions on a control instead of Page -->
<Grid utu:StatusBar.Foreground="Dark">
    <!-- This has no effect — only works on Page -->
</Grid>

<!-- WRONG: Using gradient or ImageBrush — only SolidColorBrush is supported -->
<Page utu:StatusBar.Background="{ThemeResource MyGradientBrush}" />
```

### Platform Notes
- **iOS**: Must set `UIViewControllerBasedStatusBarAppearance` to `false` in `info.plist`.
- **Desktop/Web**: These properties have no effect on non-mobile platforms.
- `Auto` and `AutoInverse` automatically update when the system or app theme changes.

---

## VisualStateManagerExtensions

**Impact: HIGH**

Bind visual states directly to a ViewModel string property using the `utu:VisualStateManagerExtensions.States` attached property. **Critical**: attach `States` to the **first child** of the element containing the `VisualStateManager.VisualStateGroups`, not to the element with the groups itself.

### CORRECT
```xml
<Grid>
    <!-- VisualStateGroups defined on the Grid -->
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup x:Name="LoadingStates">
            <VisualState x:Name="Loading">
                <VisualState.Setters>
                    <Setter Target="LoadingIndicator.Visibility" Value="Visible" />
                    <Setter Target="ContentPanel.Visibility" Value="Collapsed" />
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="Loaded">
                <VisualState.Setters>
                    <Setter Target="LoadingIndicator.Visibility" Value="Collapsed" />
                    <Setter Target="ContentPanel.Visibility" Value="Visible" />
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="Error">
                <VisualState.Setters>
                    <Setter Target="LoadingIndicator.Visibility" Value="Collapsed" />
                    <Setter Target="ContentPanel.Visibility" Value="Collapsed" />
                    <Setter Target="ErrorPanel.Visibility" Value="Visible" />
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>

    <!-- States attached to FIRST CHILD, bound to VM string property -->
    <StackPanel utu:VisualStateManagerExtensions.States="{Binding CurrentState}">
        <ProgressRing x:Name="LoadingIndicator" IsActive="True" Visibility="Collapsed" />
        <Grid x:Name="ContentPanel" Visibility="Collapsed">
            <!-- Content here -->
        </Grid>
        <Grid x:Name="ErrorPanel" Visibility="Collapsed">
            <TextBlock Text="Something went wrong" />
        </Grid>
    </StackPanel>
</Grid>
```

```csharp
// ViewModel: string property matching VisualState x:Name values
public string CurrentState { get; set; } = "Loading";

// Set to "Loaded" or "Error" based on async result
CurrentState = "Loaded";
```

### WRONG
```xml
<!-- WRONG: States attached to the SAME element as VisualStateGroups -->
<Grid utu:VisualStateManagerExtensions.States="{Binding CurrentState}">
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup>
            <VisualState x:Name="Loading" />
            <VisualState x:Name="Loaded" />
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>
    <!-- Content... -->
</Grid>
<!-- FIX: Move States to the first child element of the Grid -->

<!-- WRONG: States value doesn't match any VisualState x:Name -->
<!-- The string must exactly match a defined VisualState name -->
```

---

## SystemThemeHelper

**Impact: MEDIUM**

Detect and respond to OS theme changes, or programmatically set the app or element theme.

### CORRECT
```csharp
using Uno.Toolkit.UI;

// Get current OS theme
var currentTheme = SystemThemeHelper.GetCurrentOsTheme();
// Returns ApplicationTheme.Light or ApplicationTheme.Dark

// Set the entire application theme
SystemThemeHelper.SetApplicationTheme(ApplicationTheme.Dark);

// Set theme for a specific element subtree
SystemThemeHelper.SetRootTheme(myPage, ElementTheme.Dark);
```

### WRONG
```csharp
// WRONG: Using Application.Current.RequestedTheme directly to detect OS theme
var theme = Application.Current.RequestedTheme;
// This returns the app's requested theme, not necessarily the OS theme
// FIX: Use SystemThemeHelper.GetCurrentOsTheme()

// WRONG: Setting RequestedTheme on Application after initialization
Application.Current.RequestedTheme = ApplicationTheme.Dark;
// This throws on some platforms after app launch
// FIX: Use SystemThemeHelper.SetApplicationTheme(ApplicationTheme.Dark)
```

---

## References

- [Lightweight Styling](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/lightweight-styling.html)
- [ResourceExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/resource-extensions.html)
- [StatusBarExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/StatusBar-extensions.html)
- [VisualStateManagerExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/VisualStateManager-extensions.html)
- [SystemThemeHelper](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/SystemThemeHelper.html)
