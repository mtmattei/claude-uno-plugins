# 01 — Installation, Setup, and Configuration

**Impact**: HIGH — incorrect setup causes missing Toolkit styles, build failures, or duplicate resource keys with no clear error.

---

## Installation via UnoFeatures

Add to your `.csproj` inside `<PropertyGroup>`:

```xml
<!-- Toolkit only (rare — typically paired with a theme) -->
<UnoFeatures>Toolkit</UnoFeatures>

<!-- Material + Toolkit (recommended for most apps) -->
<UnoFeatures>Material;Toolkit</UnoFeatures>

<!-- Cupertino + Toolkit (iOS-style apps) -->
<UnoFeatures>Cupertino;Toolkit</UnoFeatures>

<!-- Full stack: Material + Toolkit + MVUX + Navigation -->
<UnoFeatures>Material;Toolkit;MVUX;Navigation</UnoFeatures>
```

> This replaces manual NuGet references. The Uno.Sdk resolves the correct packages automatically.

## CLI Scaffolding

```bash
# New app with recommended defaults (includes Material + Toolkit + MVUX)
dotnet new unoapp -preset recommended -o MyApp

# Explicit theme selection
dotnet new unoapp -theme material -o MyApp

# Cupertino theme
dotnet new unoapp -theme cupertino -o MyApp
```

---

## App.xaml Theme Setup

### MaterialToolkitTheme (Material + Toolkit)

**Impact**: CRITICAL — this is the correct single-declaration approach.

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
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <um:MaterialTheme xmlns:um="using:Uno.Material" />
            <ut:ToolkitResources xmlns:ut="using:Uno.Toolkit.UI" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

> **Rule**: If your project references `Uno.Toolkit.UI`, always use `MaterialToolkitTheme`. Never layer `MaterialTheme` + `ToolkitResources` separately — it causes duplicate resource keys and unpredictable style resolution.

### With Color and Font Overrides

```xml
<utu:MaterialToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Material"
    ColorOverrideSource="ms-appx:///Styles/ColorPaletteOverride.xaml"
    FontOverrideSource="ms-appx:///Styles/FontOverride.xaml" />
```

### With Uno.Extensions Hosting

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

## CupertinoToolkitTheme (Cupertino + Toolkit)

For iOS-style apps using Cupertino theme:

```xml
<!-- CORRECT — Cupertino + Toolkit in one declaration -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <utu:CupertinoToolkitTheme xmlns:utu="using:Uno.Toolkit.UI.Cupertino" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

```xml
<!-- WRONG — separate declarations cause duplicates -->
<uc:CupertinoTheme xmlns:uc="using:Uno.Cupertino" />
<ut:ToolkitResources xmlns:ut="using:Uno.Toolkit.UI" />
```

> Same rule applies: never layer `CupertinoTheme` + `ToolkitResources`. Use `CupertinoToolkitTheme`.

---

## C# Markup Setup

**Impact**: MEDIUM — C# Markup requires an additional NuGet and builder call.

### NuGet Package

The `Uno.Toolkit.WinUI.Material.Markup` package (or `Uno.Toolkit.WinUI.Cupertino.Markup`) provides fluent extensions.

With `UnoFeatures`, this is resolved automatically when you have `Material;Toolkit` and C# Markup enabled.

### App.cs Configuration

```csharp
// CORRECT — C# Markup with Material Toolkit
using Uno.Toolkit.UI.Material;

private IHost Host { get; set; }

protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args)
        .Configure(host => host
            .UseMaterialToolkit(
                colorOverride: new ResourceDictionary
                {
                    Source = new Uri("ms-appx:///Styles/ColorPaletteOverride.xaml")
                },
                fontOverride: new ResourceDictionary
                {
                    Source = new Uri("ms-appx:///Styles/FontOverride.xaml")
                }
            )
        );
    // ...
}
```

```csharp
// WRONG — calling UseMaterial() and UseToolkit() separately
.Configure(host => host
    .UseMaterial()
    .UseToolkit()
)
```

> Use `.UseMaterialToolkit()` (single call) or `.UseCupertinoToolkit()` for Cupertino.

---

## NuGet Packages Reference

| Package | Purpose | When to Use |
|---------|---------|-------------|
| `Uno.Toolkit.WinUI` | Core Toolkit controls and extensions | Always (auto via UnoFeatures) |
| `Uno.Toolkit.WinUI.Material` | Material-themed Toolkit styles | `MaterialToolkitTheme` (auto via UnoFeatures) |
| `Uno.Toolkit.WinUI.Cupertino` | Cupertino-themed Toolkit styles | `CupertinoToolkitTheme` (auto via UnoFeatures) |
| `Uno.Toolkit.WinUI.Material.Markup` | C# Markup extensions for Material Toolkit | C# Markup projects (auto via UnoFeatures) |
| `Uno.Toolkit.WinUI.Cupertino.Markup` | C# Markup extensions for Cupertino Toolkit | C# Markup projects (auto via UnoFeatures) |

> **Note**: With the Uno.Sdk and `<UnoFeatures>`, you rarely need to reference these packages manually. The Sdk resolves them based on your feature flags.

---

## XAML Namespaces Reference

| Prefix | Namespace | Used For |
|--------|-----------|----------|
| `utu` | `using:Uno.Toolkit.UI` | All Toolkit controls and extensions (AutoLayout, SafeArea, TabBar, CommandExtensions, etc.) |
| `utu` | `using:Uno.Toolkit.UI.Material` | `MaterialToolkitTheme` declaration in App.xaml |
| `utu` | `using:Uno.Toolkit.UI.Cupertino` | `CupertinoToolkitTheme` declaration in App.xaml |

> **Important**: Within page XAML, always use `xmlns:utu="using:Uno.Toolkit.UI"` for controls and extensions. The `.Material` and `.Cupertino` namespaces are only for theme declarations in `App.xaml`.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Build error: Toolkit types not found | Missing `Toolkit` in `<UnoFeatures>` | Add `Toolkit` (or `Material;Toolkit`) to `.csproj` |
| Toolkit controls unstyled (plain WinUI appearance) | Using `MaterialTheme` instead of `MaterialToolkitTheme` | Switch to `MaterialToolkitTheme` in App.xaml |
| Duplicate resource key warnings at runtime | Layering `MaterialTheme` + `ToolkitResources` separately | Replace with single `MaterialToolkitTheme` |
| XAML parse error on `utu:AutoLayout` | Wrong namespace — using `Uno.Toolkit` instead of `Uno.Toolkit.UI` | Use `xmlns:utu="using:Uno.Toolkit.UI"` |
| C# Markup: `.UseMaterialToolkit()` not found | Missing Markup NuGet package | Ensure `Material;Toolkit` in UnoFeatures with C# Markup enabled |
| Styles not applying after upgrading Uno version | Package version mismatch | Run `dotnet restore` and verify all Toolkit packages share the same version |

---

## Reference Links

- [Uno Toolkit Getting Started](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/getting-started.html)
- [Uno Toolkit Material Setup](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/material-getting-started.html)
- [Uno.Sdk UnoFeatures](https://platform.uno/docs/articles/features/uno-sdk.html)
- [dotnet new unoapp Options](https://platform.uno/docs/articles/get-started-dotnet-new.html)
