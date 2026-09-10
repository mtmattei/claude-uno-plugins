# 02 — Color System and Customization

**Impact**: HIGH — wrong color resource names cause runtime blanks with no compile-time warning. Using `StaticResource` instead of `ThemeResource` breaks dark mode silently.

---

## MD3 Color Roles — Full Reference

### Primary Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Primary | `PrimaryBrush` | Key components: FAB, prominent buttons, active states |
| OnPrimary | `OnPrimaryBrush` | Text/icons on Primary surfaces |
| PrimaryContainer | `PrimaryContainerBrush` | Standout fill for containers needing less emphasis than Primary |
| OnPrimaryContainer | `OnPrimaryContainerBrush` | Text/icons on PrimaryContainer |
| InversePrimary | `InversePrimaryBrush` | Primary color used against inverse surfaces |

### Secondary Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Secondary | `SecondaryBrush` | Less prominent components: filter chips, secondary actions |
| OnSecondary | `OnSecondaryBrush` | Text/icons on Secondary surfaces |
| SecondaryContainer | `SecondaryContainerBrush` | Fill for secondary containers |
| OnSecondaryContainer | `OnSecondaryContainerBrush` | Text/icons on SecondaryContainer |

### Tertiary Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Tertiary | `TertiaryBrush` | Contrast accents for balancing primary/secondary |
| OnTertiary | `OnTertiaryBrush` | Text/icons on Tertiary surfaces |
| TertiaryContainer | `TertiaryContainerBrush` | Fill for tertiary containers |
| OnTertiaryContainer | `OnTertiaryContainerBrush` | Text/icons on TertiaryContainer |

### Error Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Error | `ErrorBrush` | Error indicators, destructive actions |
| OnError | `OnErrorBrush` | Text/icons on Error surfaces |
| ErrorContainer | `ErrorContainerBrush` | Fill for error container areas |
| OnErrorContainer | `OnErrorContainerBrush` | Text/icons on ErrorContainer |

### Surface Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Surface | `SurfaceBrush` | Default background for cards, sheets, dialogs |
| OnSurface | `OnSurfaceBrush` | Primary text and icons on Surface |
| SurfaceVariant | `SurfaceVariantBrush` | Differentiated surface (e.g., input fields, chips) |
| OnSurfaceVariant | `OnSurfaceVariantBrush` | Secondary text/icons on surfaces |
| SurfaceTint | `SurfaceTintBrush` | Tint overlay for elevation |
| InverseSurface | `InverseSurfaceBrush` | Surface for snackbars, tooltips (contrasts main surface) |
| InverseOnSurface | `InverseOnSurfaceBrush` | Text/icons on InverseSurface |

### Background Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Background | `BackgroundBrush` | App window/page background |
| OnBackground | `OnBackgroundBrush` | Text/icons on Background |

### Outline Group

| Role | Brush Resource | Usage |
|------|---------------|-------|
| Outline | `OutlineBrush` | Important boundaries, input field outlines |
| OutlineVariant | `OutlineVariantBrush` | Decorative borders, dividers |

---

## Brush Resource Naming Convention

Pattern: `{Role}Brush`

```xml
<!-- CORRECT — Material brush resources -->
<Border Background="{ThemeResource SurfaceBrush}" />
<TextBlock Foreground="{ThemeResource OnSurfaceBrush}" />
<Border BorderBrush="{ThemeResource OutlineBrush}" />
<Button Background="{ThemeResource PrimaryBrush}"
        Foreground="{ThemeResource OnPrimaryBrush}" />

<!-- WRONG — WinUI system brushes do NOT exist in Material theme -->
<Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}" />
<TextBlock Foreground="{ThemeResource SystemControlForegroundBaseHighBrush}" />
```

---

## ThemeResource vs StaticResource

**Impact**: HIGH — using `StaticResource` for theme-aware brushes silently breaks dark/light mode switching.

```xml
<!-- CORRECT — resolves per-theme at runtime, updates on theme change -->
<Border Background="{ThemeResource PrimaryBrush}" />

<!-- WRONG — locks to the value at first resolution, ignores theme changes -->
<Border Background="{StaticResource PrimaryBrush}" />
```

**Rule**: Always use `{ThemeResource}` for Material color brushes. Only use `{StaticResource}` for non-theme-dependent resources (e.g., converters, data templates, typography styles).

---

## Opacity Resources

| Resource Key | Default Value | Usage |
|-------------|---------------|-------|
| `HoverOpacity` | 0.08 | Hover state overlay |
| `FocusedOpacity` | 0.12 | Focus state overlay |
| `PressedOpacity` | 0.12 | Pressed state overlay |
| `DraggedOpacity` | 0.16 | Dragged state overlay |
| `DisabledOpacity` | 0.38 | Disabled elements |
| `SelectedOpacity` | 0.08 | Selected state overlay |

These are applied as state layer overlays on top of the base color. They can be overridden via lightweight styling.

---

## Color Customization — ColorPaletteOverride.xaml

Create a `ResourceDictionary` with your custom colors:

```xml
<!-- Styles/ColorPaletteOverride.xaml -->
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <!-- Light theme overrides -->
    <ResourceDictionary.ThemeDictionaries>
        <ResourceDictionary x:Key="Light">
            <Color x:Key="PrimaryColor">#6750A4</Color>
            <Color x:Key="OnPrimaryColor">#FFFFFF</Color>
            <Color x:Key="PrimaryContainerColor">#EADDFF</Color>
            <Color x:Key="OnPrimaryContainerColor">#21005D</Color>
            <Color x:Key="SecondaryColor">#625B71</Color>
            <Color x:Key="OnSecondaryColor">#FFFFFF</Color>
            <Color x:Key="SurfaceColor">#FFFBFE</Color>
            <Color x:Key="OnSurfaceColor">#1C1B1F</Color>
            <Color x:Key="ErrorColor">#B3261E</Color>
            <Color x:Key="OnErrorColor">#FFFFFF</Color>
            <!-- ... additional roles ... -->
        </ResourceDictionary>

        <ResourceDictionary x:Key="Dark">
            <Color x:Key="PrimaryColor">#D0BCFF</Color>
            <Color x:Key="OnPrimaryColor">#381E72</Color>
            <Color x:Key="PrimaryContainerColor">#4F378B</Color>
            <Color x:Key="OnPrimaryContainerColor">#EADDFF</Color>
            <Color x:Key="SurfaceColor">#1C1B1F</Color>
            <Color x:Key="OnSurfaceColor">#E6E1E5</Color>
            <!-- ... additional roles ... -->
        </ResourceDictionary>
    </ResourceDictionary.ThemeDictionaries>

</ResourceDictionary>
```

Reference it in `App.xaml`:

```xml
<utu:MaterialToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Material"
    ColorOverrideSource="ms-appx:///Styles/ColorPaletteOverride.xaml" />
```

> **Note**: Color resource keys use `{Role}Color` (e.g., `PrimaryColor`), while the generated brushes are `{Role}Brush`. You define `Color` values; the theme generates the corresponding `SolidColorBrush` resources automatically.

---

## DSP Import Workflow (Material Theme Builder)

**Impact**: MEDIUM — automates color generation but requires build-time tooling.

### Steps

1. Go to [Material Theme Builder](https://m3.material.io/theme-builder) and design your palette
2. Export as **DSP (JSON)** format
3. Place the exported JSON file in your project's `Styles/` folder
4. Add DSP support to `<UnoFeatures>`:

```xml
<UnoFeatures>Material;Toolkit;Dsp</UnoFeatures>
```

5. The `Uno.Dsp.Tasks` MSBuild task auto-generates the XAML `ResourceDictionary` at build time
6. The generated file is placed alongside the JSON source and picked up by the theme automatically

```
MyApp/
  Styles/
    MaterialTheme.json     ← exported from Material Theme Builder
    MaterialTheme.xaml     ← auto-generated at build (do not hand-edit)
```

> **Do not hand-edit the generated `.xaml` file** — it will be overwritten on next build. Edit the `.json` source or re-export from Material Theme Builder.

---

## C# Markup Color Override

When using C# Markup instead of XAML, override colors programmatically:

```csharp
using Uno.Material;
using Uno.Themes.Markup;

// In your App.cs or host builder
new MaterialToolkitTheme()
    .ColorOverride(
        builder => builder
            .Add<Color>(Theme.Colors.Primary.Default.Key, light: "#6750A4", dark: "#D0BCFF")
            .Add<Color>(Theme.Colors.OnPrimary.Default.Key, light: "#FFFFFF", dark: "#381E72")
            .Add<Color>(Theme.Colors.Surface.Default.Key, light: "#FFFBFE", dark: "#1C1B1F")
            .Add<Color>(Theme.Colors.OnSurface.Default.Key, light: "#1C1B1F", dark: "#E6E1E5")
    );
```

Key types:
- `Theme.Colors.Primary.Default.Key` — strongly-typed key for Primary color
- `Theme.Colors.Secondary.Default.Key` — Secondary color
- Pattern: `Theme.Colors.{Role}.Default.Key`

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Using WinUI system brushes (`ApplicationPageBackgroundThemeBrush`, `SystemControlForegroundBaseHighBrush`) | Brush not found at runtime — blank or transparent | Use Material equivalents: `SurfaceBrush`, `OnSurfaceBrush` |
| `{StaticResource PrimaryBrush}` for theme-aware brush | Light/dark mode switch does not update the color | Use `{ThemeResource PrimaryBrush}` |
| Defining `PrimaryBrush` instead of `PrimaryColor` in override XAML | Override ignored — theme expects `Color` resources, not `Brush` | Use `{Role}Color` keys (e.g., `PrimaryColor`) in your override dictionary |
| Missing `ThemeDictionaries` wrapper in override file | Colors only apply to one theme (usually Light) | Wrap in `ResourceDictionary.ThemeDictionaries` with `Light` and `Dark` keys |
| Hand-editing DSP-generated XAML | Changes lost on next build | Edit the source `.json` or re-export from Material Theme Builder |

---

## Reference Links

- [Material Colors & Brushes](https://platform.uno/docs/articles/external/uno.themes/doc/material-colors.html)
- [Color Customization](https://platform.uno/docs/articles/external/uno.themes/doc/material-colors-customization.html)
- [DSP Import](https://platform.uno/docs/articles/external/uno.themes/doc/dsp-import.html)
- [Material Theme Builder](https://m3.material.io/theme-builder)
- [MD3 Color System Spec](https://m3.material.io/styles/color/system)
