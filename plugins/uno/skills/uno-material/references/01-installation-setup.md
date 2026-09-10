# 01 — Installation, Setup, and Migration

**Impact**: HIGH — incorrect setup causes blank pages, missing styles, or build failures with no clear error.

---

## Installation via UnoFeatures

Add to your `.csproj` inside `<PropertyGroup>`:

```xml
<!-- Material only -->
<UnoFeatures>Material</UnoFeatures>

<!-- Material + Toolkit (recommended for most apps) -->
<UnoFeatures>Material;Toolkit</UnoFeatures>

<!-- Material + Toolkit + DSP color import -->
<UnoFeatures>Material;Toolkit;Dsp</UnoFeatures>
```

> This replaces manual NuGet references. The Uno.Sdk resolves the correct packages automatically.

## CLI Scaffolding

```bash
# New app with Material theme
dotnet new unoapp -theme material -o MyApp

# With preset (recommended defaults including Material + Toolkit + MVUX)
dotnet new unoapp -preset recommended -o MyApp
```

---

## App.xaml Setup

### MaterialTheme (without Toolkit)

```xml
<!-- CORRECT — Material only -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <um:MaterialTheme xmlns:um="using:Uno.Material" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

### MaterialToolkitTheme (with Toolkit)

```xml
<!-- CORRECT — Material + Toolkit in one declaration -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <utu:MaterialToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Material" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

```xml
<!-- WRONG — never layer these separately -->
<um:MaterialTheme xmlns:um="using:Uno.Material" />
<ut:ToolkitResources xmlns:ut="using:Uno.Toolkit.UI" />
```

### Color and Font Override Sources

Override defaults by passing constructor parameters:

```xml
<utu:MaterialToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Material"
    ColorOverrideSource="ms-appx:///Styles/ColorPaletteOverride.xaml"
    FontOverrideSource="ms-appx:///Styles/FontOverride.xaml" />
```

### Host Property (for Uno.Extensions hosting)

When using `IHostBuilder`, declare the theme via the `Host` property:

```xml
<utu:MaterialToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Material"
    x:Name="MaterialTheme">
    <utu:MaterialToolkitTheme.Host>
        <!-- IHost instance set from code-behind or via x:Bind -->
    </utu:MaterialToolkitTheme.Host>
</utu:MaterialToolkitTheme>
```

---

## XAML Namespaces Reference

| Prefix | Namespace | Used For |
|--------|-----------|----------|
| `um` | `using:Uno.Material` | `MaterialTheme` (without Toolkit) |
| `utu` | `using:Uno.Toolkit.UI.Material` | `MaterialToolkitTheme` (with Toolkit) |
| `ut` | `using:Uno.Themes` | Control extensions (Icon, Elevation) since v5 |

---

## Migration Guides

### v1 to v2 — Material Design 2 to MD3

**Impact**: HIGH — resource key names changed across the board.

| Change | Action |
|--------|--------|
| `MaterialResources` class renamed | Use `MaterialResourcesV1` to stay on v1, or adopt new MD3 naming |
| Color keys renamed to MD3 roles | `PrimaryColor` → `PrimaryBrush`, follow MD3 role names |
| Typography keys renamed | `Headline1` → `DisplayLarge`, `Body1` → `BodyLarge` |
| Button style names changed | `ContainedButtonStyle` → `FilledButtonStyle` |

```xml
<!-- v1 (legacy) — opt-in to keep old keys -->
<um:MaterialResourcesV1 xmlns:um="using:Uno.Material" />

<!-- v2 (recommended) — new MD3 keys -->
<um:MaterialTheme xmlns:um="using:Uno.Material" />
```

### v2 to v3 — Lightweight Styling Support

**Impact**: MEDIUM — new capabilities, some key renames.

| Change | Action |
|--------|--------|
| Lightweight styling enabled | Override resource keys at App/Page/Control level without ControlTemplate copies |
| Some resource keys renamed for consistency | Check release notes for specific renames |
| New control extension properties | `ut:ControlExtensions.Icon` property added |

### v3 to v5 — Namespace and Opacity Changes

**Impact**: HIGH — build-breaking namespace change.

| Change | Action |
|--------|--------|
| ControlExtensions moved from `Uno.Material` to `Uno.Themes` | Change `xmlns:um="using:Uno.Material"` to `xmlns:ut="using:Uno.Themes"` for extensions |
| Opacity resource values changed | `HoverOpacity`, `FocusedOpacity`, `PressedOpacity` values updated to MD3 spec |
| Attached property namespace | `um:ControlExtensions.Icon` → `ut:ControlExtensions.Icon` |

```xml
<!-- CORRECT (v5+) -->
<Button Content="Save"
        xmlns:ut="using:Uno.Themes"
        ut:ControlExtensions.Icon="{StaticResource SaveIcon}" />

<!-- WRONG (pre-v5 syntax, will not compile in v5+) -->
<Button Content="Save"
        xmlns:um="using:Uno.Material"
        um:ControlExtensions.Icon="{StaticResource SaveIcon}" />
```

### v5.1 — TextBox Header/PlaceholderText Behavior

**Impact**: MEDIUM — visual regression if unaddressed.

| Change | Action |
|--------|--------|
| `Header` property now renders above the TextBox (not as floating label) | Use `PlaceholderText` for floating label behavior |
| `PlaceholderText` is the floating label in Material TextBox | Move text from `Header` to `PlaceholderText` if you want the MD3 floating label |

```xml
<!-- CORRECT (v5.1+) — floating label -->
<TextBox PlaceholderText="Email address"
         Style="{StaticResource OutlinedTextBoxStyle}" />

<!-- This now renders a static header ABOVE the TextBox, not a floating label -->
<TextBox Header="Email address"
         Style="{StaticResource OutlinedTextBoxStyle}" />
```

### v5.4 — Material Prefix Removal

**Impact**: MEDIUM — widespread key renames, but non-breaking (old keys may still resolve via aliases).

| Change | Action |
|--------|--------|
| `Material` prefix removed from many resource keys | `MaterialFilledButtonStyle` → `FilledButtonStyle` |
| Style keys simplified | `MaterialOutlinedTextBoxStyle` → `OutlinedTextBoxStyle` |
| Brush keys simplified | Resource keys remain `{Role}Brush` pattern (unchanged) |

```xml
<!-- CORRECT (v5.4+) -->
<Button Style="{StaticResource FilledButtonStyle}" />
<TextBox Style="{StaticResource OutlinedTextBoxStyle}" />

<!-- WRONG (pre-v5.4, may still work via aliases but deprecated) -->
<Button Style="{StaticResource MaterialFilledButtonStyle}" />
<TextBox Style="{StaticResource MaterialOutlinedTextBoxStyle}" />
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Blank page on startup | Missing or wrong theme declaration in App.xaml | Verify `MaterialToolkitTheme` is in `MergedDictionaries` |
| `XamlParseException` referencing theme type | `Material` not in `<UnoFeatures>` | Add `Material` (or `Material;Toolkit`) to `.csproj` |
| Styles not applying to Toolkit controls | Using `MaterialTheme` instead of `MaterialToolkitTheme` | Switch to `MaterialToolkitTheme` |
| Dark mode colors not updating | Using `{StaticResource}` for brushes | Switch to `{ThemeResource}` |
| ControlExtensions build error | Wrong namespace after v5 migration | Use `xmlns:ut="using:Uno.Themes"` |

---

## Reference Links

- [Material Getting Started](https://platform.uno/docs/articles/external/uno.themes/doc/material-getting-started.html)
- [Migration Guide](https://platform.uno/docs/articles/external/uno.themes/doc/migration.html)
- [Uno.Sdk UnoFeatures](https://platform.uno/docs/articles/features/uno-sdk.html)
- [dotnet new unoapp options](https://platform.uno/docs/articles/get-started-dotnet-new.html)
