---
name: uno-material
description: "Uno Material Design 3 theming: installation, MD3 color system, typography, button/TextBox/FAB styles, control extensions, lightweight styling, C# Markup integration, and version migration. Use when: (1) Installing or configuring Material theme, (2) Customizing colors or fonts, (3) Applying Material button/TextBox/FAB styles, (4) Using lightweight styling overrides, (5) Migrating between Material versions, (6) Debugging blank pages from wrong resource keys. Do NOT use for: navigation (see uno-navigation), MVUX patterns (see mvux), general XAML layout (see winui-xaml)."
license: "MIT"
metadata:
  version: "1.0.0"
  author: "Uno Platform"
  category: "uno-platform-material"
  tags: [material, theming, md3, colors, typography, styles, uno-platform]
---

# Uno Material Design 3 — Hub Skill

## Quick Reference (10 Rules)

| # | Rule | Impact |
|---|------|--------|
| 1 | **csproj**: Add `Material` to `<UnoFeatures>` — with Toolkit: `Material;Toolkit` (two separate features) | Setup will fail without it |
| 2 | **App.xaml**: Use `MaterialToolkitTheme` from `using:Uno.Toolkit.UI.Material` (NOT both `MaterialTheme` + `ToolkitResources` separately). Without Toolkit, use `MaterialTheme` from `using:Uno.Material` | Duplicate resources / style conflicts |
| 3 | Typography short keys: `DisplayLarge`, `HeadlineMedium`, `BodySmall` — NOT `TitleLargeTextBlockStyle` | Silent blank text |
| 4 | Wrong resource keys cause silent blank pages — no compile error | Hours of debugging |
| 5 | Color resources: `{ThemeResource PrimaryBrush}`, `{ThemeResource OnSurfaceBrush}` | Theme-switching breaks with StaticResource |
| 6 | Button styles: `FilledButtonStyle` (default), `ElevatedButtonStyle`, `OutlinedButtonStyle`, `TextButtonStyle`, `FilledTonalButtonStyle` | Wrong style name = unstyled button |
| 7 | TextBox styles: `OutlinedTextBoxStyle` (default), `FilledTextBoxStyle` | Mismatch causes plain WinUI TextBox |
| 8 | FAB styles: `FabStyle`, `SmallFabStyle`, `LargeFabStyle`, `SurfaceFabStyle`, `SecondaryFabStyle`, `TertiaryFabStyle` | Wrong FAB name = no FAB rendering |
| 9 | Lightweight styling: override resource keys at App/Page/Control level — no template redefinition needed | Avoids brittle ControlTemplate copies |
| 10 | Control extensions namespace: `xmlns:ut="using:Uno.Themes"` (NOT `Uno.Material` since v5) | Build error or silent no-op |

---

## Key Concepts

### MaterialTheme vs MaterialToolkitTheme

| Scenario | Use | Why |
|----------|-----|-----|
| Material only (no Toolkit controls) | `MaterialTheme` | Lighter footprint |
| Material + Toolkit controls (Card, TabBar, NavigationBar, etc.) | `MaterialToolkitTheme` | Single merged dictionary — replaces both `MaterialTheme` and `ToolkitResources` |

> **Rule**: If your project references `Uno.Toolkit.UI`, always use `MaterialToolkitTheme`. Never layer `MaterialTheme` + `ToolkitResources` separately — it causes duplicate resource keys and unpredictable style resolution.

### MD3 Color Roles

| Role Group | Colors |
|------------|--------|
| Primary | `Primary`, `OnPrimary`, `PrimaryContainer`, `OnPrimaryContainer`, `InversePrimary` |
| Secondary | `Secondary`, `OnSecondary`, `SecondaryContainer`, `OnSecondaryContainer` |
| Tertiary | `Tertiary`, `OnTertiary`, `TertiaryContainer`, `OnTertiaryContainer` |
| Error | `Error`, `OnError`, `ErrorContainer`, `OnErrorContainer` |
| Surface | `Surface`, `OnSurface`, `SurfaceVariant`, `OnSurfaceVariant`, `SurfaceTint`, `InverseSurface`, `InverseOnSurface` |
| Background | `Background`, `OnBackground` |
| Outline | `Outline`, `OutlineVariant` |

Brush resource naming pattern: `{Role}Brush` (e.g., `PrimaryBrush`, `OnSurfaceBrush`, `SecondaryContainerBrush`).

### Typography Scale

Pattern: `{Category}{Size}` where:
- **Category**: Display, Headline, Title, Body, Label, Caption
- **Size**: Large, Medium, Small

Examples: `DisplayLarge`, `HeadlineMedium`, `BodySmall`, `LabelLarge`

Applied as: `Style="{StaticResource DisplayLarge}"`

---

## Critical Patterns

### Typography Keys

```xml
<!-- CORRECT -->
<TextBlock Text="Hello" Style="{StaticResource TitleLarge}" />

<!-- WRONG — "TextBlockStyle" suffix does not exist in Material -->
<TextBlock Text="Hello" Style="{StaticResource TitleLargeTextBlockStyle}" />
```

### Color Resources

```xml
<!-- CORRECT — ThemeResource resolves at runtime per theme -->
<Border Background="{ThemeResource SurfaceBrush}" />
<TextBlock Foreground="{ThemeResource OnSurfaceBrush}" />

<!-- WRONG — WinUI system brush does not exist in Material theme -->
<Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}" />
```

### MaterialToolkitTheme Setup

```xml
<!-- CORRECT — single merged theme -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <utu:MaterialToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Material" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>

<!-- WRONG — separate MaterialTheme + ToolkitResources causes duplicates -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <um:MaterialTheme xmlns:um="using:Uno.Material" />
            <ut:ToolkitResources xmlns:ut="using:Uno.Toolkit.UI" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong resource key name (e.g., `TitleLargeTextBlockStyle`) | Silent blank page or unstyled control — no compile error | Use short keys: `TitleLarge`, `BodyMedium`, etc. |
| Missing `Material` in `<UnoFeatures>` | Build error: theme types not found | Add `Material` (or `Material;Toolkit`) to `<UnoFeatures>` |
| Using `{StaticResource}` for theme-aware brushes | Colors don't update on theme switch (light/dark) | Always use `{ThemeResource PrimaryBrush}` for Material brushes |
| ControlExtensions namespace `xmlns:um="using:Uno.Material"` | Build error or extension properties silently ignored | Since v5, use `xmlns:ut="using:Uno.Themes"` |
| Using WinUI system brushes (`ApplicationPageBackgroundThemeBrush`) | Brush not found at runtime — blank or default background | Use Material equivalents: `SurfaceBrush`, `BackgroundBrush` |
| Layering `MaterialTheme` + `ToolkitResources` separately | Duplicate keys, unpredictable style resolution | Use single `MaterialToolkitTheme` |

---

## Related Skills

| Skill | Use When |
|-------|----------|
| `uno-navigation` | Setting up region-based navigation, route registration, NavigationView |
| `mvux` | Building reactive models with IFeed/IState, FeedView, commands |
| `winui-xaml` | General XAML layout, binding, rendering, accessibility |
| `uno-csharp-markup` | Building UI in C# instead of XAML with fluent API |
| `uno-toolkit` | Using Toolkit controls (Card, TabBar, SafeArea, DrawerControl) |
| `uno-extensions-services` | Hosting, DI, authentication, HTTP, configuration |

---

## Detailed References

| # | File | Covers |
|---|------|--------|
| 1 | `references/01-installation-setup.md` | Installation, App.xaml setup, CLI scaffolding, version migration (v1-v5.4) |
| 2 | `references/02-color-system.md` | MD3 color roles, brush naming, color overrides, DSP import, C# Markup colors |
| 3 | `references/03-typography.md` | Typography scale, font customization, font override source |
| 4 | `references/04-control-styles.md` | Button, TextBox, FAB, PasswordBox styles and variants |
| 5 | `references/05-lightweight-styling-csharp.md` | Resource key overrides, control extensions, scoped theming, C# Markup |

---

## Docs Fallback

If this skill does not cover your scenario, search the official Uno Platform docs:

1. Use `uno_platform_docs_search` with queries like "Material theme setup", "MD3 color override", "Material typography"
2. Use `uno_platform_docs_fetch` to retrieve the full page content from a search result URL
3. Key doc sections:
   - Uno.Themes overview: `https://platform.uno/docs/articles/external/uno.themes/doc/material-getting-started.html`
   - Color system: `https://platform.uno/docs/articles/external/uno.themes/doc/material-colors.html`
   - Lightweight styling: `https://platform.uno/docs/articles/external/uno.themes/doc/lightweight-styling.html`
   - Migration guide: `https://platform.uno/docs/articles/external/uno.themes/doc/migration.html`
