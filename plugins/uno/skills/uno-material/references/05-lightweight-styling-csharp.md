# Lightweight Styling and C# Markup

## Impact: CRITICAL
Lightweight styling is the recommended way to customize Material control appearance without redefining entire control templates. Using the wrong resource key, wrong scope, or `StaticResource` instead of `ThemeResource` causes silent failures or broken dark/light theme switching.

---

## Concept

Lightweight styling overrides specific resource keys consumed by a control's template to change appearance (colors, fonts, corner radius, spacing) without redefining the template itself. This keeps customizations upgrade-safe and composable.

---

## Three Override Scopes

**Impact: CRITICAL**

### 1. App Level (App.xaml) -- Global override

```xml
<!-- App.xaml -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <MaterialTheme x:Name="MaterialTheme" />
        </ResourceDictionary.MergedDictionaries>

        <!-- Global: ALL FilledButtons get custom foreground -->
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="FilledButtonForeground" Color="#FFFFFF" />
                <SolidColorBrush x:Key="FilledButtonForegroundPointerOver" Color="#F0F0F0" />
                <SolidColorBrush x:Key="FilledButtonForegroundPressed" Color="#E0E0E0" />
                <SolidColorBrush x:Key="FilledButtonForegroundDisabled" Color="#A0A0A0" />
            </ResourceDictionary>
            <ResourceDictionary x:Key="Default">
                <!-- "Default" = Dark theme -->
                <SolidColorBrush x:Key="FilledButtonForeground" Color="#000000" />
                <SolidColorBrush x:Key="FilledButtonForegroundPointerOver" Color="#101010" />
                <SolidColorBrush x:Key="FilledButtonForegroundPressed" Color="#202020" />
                <SolidColorBrush x:Key="FilledButtonForegroundDisabled" Color="#606060" />
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
                <SolidColorBrush x:Key="FilledButtonBackground" Color="DarkGreen" />
            </ResourceDictionary>
            <ResourceDictionary x:Key="Default">
                <SolidColorBrush x:Key="FilledButtonBackground" Color="LightGreen" />
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Page.Resources>
```

### 3. Control Level (Control.Resources) -- Single control only

```xml
<Button Content="Special Button"
        Style="{StaticResource FilledButtonStyle}">
    <Button.Resources>
        <ResourceDictionary>
            <ResourceDictionary.ThemeDictionaries>
                <ResourceDictionary x:Key="Light">
                    <SolidColorBrush x:Key="FilledButtonBackground" Color="OrangeRed" />
                </ResourceDictionary>
                <ResourceDictionary x:Key="Default">
                    <SolidColorBrush x:Key="FilledButtonBackground" Color="DarkOrange" />
                </ResourceDictionary>
            </ResourceDictionary.ThemeDictionaries>
        </ResourceDictionary>
    </Button.Resources>
</Button>
```

---

## ThemeDictionaries Keys

**Impact: CRITICAL**

| x:Key       | Theme          |
|-------------|----------------|
| `"Light"`   | Light theme    |
| `"Default"` | Dark theme     |

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
<!-- WRONG: "Dark" is not a valid key -->
<ResourceDictionary x:Key="Dark">
    <!-- These overrides will be ignored -->
</ResourceDictionary>

<!-- WRONG: Overriding outside ThemeDictionaries (won't respond to theme changes) -->
<SolidColorBrush x:Key="FilledButtonBackground" Color="Red" />
```

---

## Resource Key Naming Convention

**Impact: CRITICAL**

Pattern: `{Style}{Property}{State}`

### States

| State          | When Applied                          |
|----------------|---------------------------------------|
| *(empty)*      | Default / rest state                  |
| `PointerOver`  | Mouse hover                           |
| `Pressed`      | Active press                          |
| `Disabled`     | `IsEnabled="False"`                   |
| `Focused`      | Keyboard focus                        |

#### CheckBox / RadioButton Additional States

| State          | Combined with        |
|----------------|----------------------|
| `Checked`      | Default, PointerOver, Pressed, Disabled |
| `Unchecked`    | Default, PointerOver, Pressed, Disabled |
| `Indeterminate`| Default, PointerOver, Pressed, Disabled |

### Examples

| Key | Meaning |
|-----|---------|
| `FilledButtonForeground` | Filled button text color at rest |
| `FilledButtonForegroundPointerOver` | Filled button text color on hover |
| `FilledButtonBackground` | Filled button background at rest |
| `FilledButtonBackgroundPressed` | Filled button background when pressed |
| `FilledButtonBackgroundDisabled` | Filled button background when disabled |
| `OutlinedTextBoxBorderBrush` | Outlined TextBox border at rest |
| `OutlinedTextBoxBorderBrushFocused` | Outlined TextBox border when focused |

### CORRECT
```xml
<SolidColorBrush x:Key="FilledButtonForegroundPointerOver" Color="White" />
```

### WRONG
```xml
<!-- WRONG: Inventing state names -->
<SolidColorBrush x:Key="FilledButtonForegroundHover" Color="White" />

<!-- WRONG: Wrong order (State before Property) -->
<SolidColorBrush x:Key="FilledButtonPointerOverForeground" Color="White" />
```

---

## ResourceExtensions.Resources (Uno Toolkit)

**Impact: HIGH**

Attach a `ResourceDictionary` directly to a `Style` for reusable lightweight-styling variants without polluting page-level resources.

### CORRECT
```xml
xmlns:ut="using:Uno.Themes"
xmlns:utu="using:Uno.Toolkit.UI"

<Style x:Key="DangerFilledButtonStyle"
       TargetType="Button"
       BasedOn="{StaticResource FilledButtonStyle}">
    <Setter Property="utu:ResourceExtensions.Resources">
        <Setter.Value>
            <ResourceDictionary>
                <ResourceDictionary.ThemeDictionaries>
                    <ResourceDictionary x:Key="Light">
                        <SolidColorBrush x:Key="FilledButtonBackground" Color="#DC2626" />
                        <SolidColorBrush x:Key="FilledButtonBackgroundPointerOver" Color="#B91C1C" />
                        <SolidColorBrush x:Key="FilledButtonBackgroundPressed" Color="#991B1B" />
                    </ResourceDictionary>
                    <ResourceDictionary x:Key="Default">
                        <SolidColorBrush x:Key="FilledButtonBackground" Color="#EF4444" />
                        <SolidColorBrush x:Key="FilledButtonBackgroundPointerOver" Color="#DC2626" />
                        <SolidColorBrush x:Key="FilledButtonBackgroundPressed" Color="#B91C1C" />
                    </ResourceDictionary>
                </ResourceDictionary.ThemeDictionaries>
            </ResourceDictionary>
        </Setter.Value>
    </Setter>
</Style>

<!-- Usage -->
<Button Content="Delete" Style="{StaticResource DangerFilledButtonStyle}" />
```

### WRONG
```xml
<!-- WRONG: Applying ResourceExtensions directly on the control (use Button.Resources instead) -->
<Button Content="Delete"
        utu:ResourceExtensions.Resources="{StaticResource SomeDictionary}" />
```

---

## Controls Supporting Lightweight Styling

**Impact: MEDIUM**

Material controls: Button, TextBox, PasswordBox, CheckBox, RadioButton, ComboBox, Slider, ToggleSwitch, ToggleButton, FAB, NavigationView, DatePicker, TimePicker, CalendarDatePicker, ProgressBar, ProgressRing

Toolkit controls: Card, Chip, ChipGroup, NavigationBar, TabBar, Divider, DrawerControl

---

## C# Markup Integration

**Impact: HIGH**

### Initialization with Material Theme

#### CORRECT
```csharp
// In App.cs
this.Resources
    .Build(r => r
        .Merged(new MaterialTheme()
            .ColorOverride(colorOverride)
            .FontOverride(fontOverride)));
```

Or using the extension method:

```csharp
// In App.cs with Uno.Extensions host builder
private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes) { ... }

protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var builder = this.CreateBuilder(args)
        .UseMaterialTheme();  // Registers Material resources
}
```

### Theme Style References

#### CORRECT
```csharp
using Uno.Material;
using Uno.Themes.Markup;

new Button()
    .Content("Save")
    .Style(Theme.Button.Styles.Filled)

new Button()
    .Content("Cancel")
    .Style(Theme.Button.Styles.Outlined)

new TextBox()
    .Style(Theme.TextBox.Styles.Outlined)
    .Header("Name")
    .PlaceholderText("Enter name")
```

### Theme Brush References

#### CORRECT
```csharp
using Uno.Themes.Markup;

new Border()
    .Background(Theme.Brushes.Surface.Default)

new TextBlock()
    .Text("Hello")
    .Foreground(Theme.Brushes.OnSurface.Default)
```

### Per-Control Lightweight Styling Override

#### CORRECT
```csharp
new Button()
    .Content("Danger")
    .Style(Theme.Button.Styles.Filled)
    .Resources(config => config
        .Add("FilledButtonBackground", new SolidColorBrush(Colors.Red))
        .Add("FilledButtonBackgroundPointerOver", new SolidColorBrush(Colors.DarkRed))
        .Add("FilledButtonBackgroundPressed", new SolidColorBrush(Colors.Maroon)))
```

#### WRONG
```csharp
// WRONG: Setting Background directly bypasses lightweight styling and breaks states
new Button()
    .Content("Danger")
    .Style(Theme.Button.Styles.Filled)
    .Background(new SolidColorBrush(Colors.Red))

// WRONG: Using wrong resource key format
new Button()
    .Resources(config => config
        .Add("ButtonBackground", new SolidColorBrush(Colors.Red)))
```

---

## StaticResource vs ThemeResource

**Impact: CRITICAL**

### CORRECT
```xml
<!-- ThemeResource: responds to light/dark theme changes at runtime -->
<TextBlock Foreground="{ThemeResource OnSurfaceBrush}" />

<!-- StaticResource: OK for styles (they don't change with theme) -->
<Button Style="{StaticResource FilledButtonStyle}" />
```

### WRONG
```xml
<!-- WRONG: StaticResource for brushes won't update when theme changes -->
<TextBlock Foreground="{StaticResource OnSurfaceBrush}" />

<!-- WRONG: ThemeResource for styles is unnecessary (but not harmful) -->
<Button Style="{ThemeResource FilledButtonStyle}" />
```

**Rule of thumb:**
- `{StaticResource ...}` for **Styles** (assigned once, don't change with theme)
- `{ThemeResource ...}` for **Brushes, Colors, and theme-aware values** (switch with light/dark)

---

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Using `StaticResource` for brushes | CRITICAL | Use `ThemeResource` for all brush/color references |
| Using `x:Key="Dark"` in ThemeDictionaries | CRITICAL | Use `x:Key="Default"` for dark theme |
| Wrong key order: `FilledButtonPointerOverForeground` | CRITICAL | Correct order: `FilledButtonForegroundPointerOver` |
| Setting `Background` property directly on styled control | HIGH | Use lightweight styling resource keys instead |
| Overriding at App level when Page level suffices | MEDIUM | Scope overrides to the narrowest level needed |
| Missing state overrides (only overriding rest state) | HIGH | Override all states: default, PointerOver, Pressed, Disabled |
| Inventing state names like `Hover` or `Active` | HIGH | Use exact states: PointerOver, Pressed, Disabled, Focused |
| Forgetting ThemeDictionaries wrapper | HIGH | Always wrap theme-aware overrides in ThemeDictionaries |

---

## References

- [Uno Material Lightweight Styling](https://platform.uno/docs/articles/external/uno.themes/doc/material-lightweight-styling.html)
- [Uno Toolkit ResourceExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/resource-extensions.html)
- [Uno C# Markup Material](https://platform.uno/docs/articles/external/uno.themes/doc/material-csharp-markup.html)
- [Material Design 3 Theming](https://m3.material.io/foundations/customization)
- [WinUI ThemeResource vs StaticResource](https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-resource-dictionary)
