---
name: uno-toolkit
description: "Uno Toolkit UI controls and helpers: AutoLayout, SafeArea, Card, Chip, TabBar, NavigationBar, DrawerControl, ShadowContainer, and extensions (CommandExtensions, InputExtensions, ResponsiveExtension, ItemsRepeaterExtensions, StatusBarExtensions). Use when: (1) Building responsive layouts with AutoLayout or SafeArea, (2) Using Card/Chip/TabBar/NavigationBar controls, (3) Attaching commands to control events, (4) Implementing form input UX, (5) Adding selection/incremental loading to ItemsRepeater, (6) Lightweight styling Toolkit controls. Do NOT use for: Material theme colors/typography (see uno-material), navigation routing (see uno-navigation), MVUX patterns (see mvux)."
license: "MIT"
metadata:
  version: "1.0.0"
  author: "Uno Platform"
  category: "uno-platform-toolkit"
  tags: [toolkit, controls, extensions, autolayout, safearea, tabbar, uno-platform]
---

# Uno Toolkit — Hub Skill

Controls, layout helpers, and attached extensions for cross-platform Uno Platform apps. The Toolkit adds Figma-aligned layout (AutoLayout), mobile-safe input handling (SafeArea), Material-styled containers (Card, Chip, Drawer), navigation chrome (TabBar, NavigationBar), and XAML-attached behaviors (CommandExtensions, InputExtensions, ResponsiveExtension).

## Quick Reference (10 Rules)

| # | Rule | Impact |
|---|------|--------|
| 1 | **csproj**: Add `Material;Toolkit` to `<UnoFeatures>` (two separate features). **App.xaml**: Use `MaterialToolkitTheme` (NOT separate `MaterialTheme` + `ToolkitResources`) | Duplicate resources / style conflicts |
| 2 | XAML namespace for **controls**: `xmlns:utu="using:Uno.Toolkit.UI"`. Theme declaration in **App.xaml** uses a different namespace: `using:Uno.Toolkit.UI.Material` (or `.Cupertino`) | Build error if wrong namespace |
| 3 | AutoLayout: Figma-like layout — set `Orientation`, `Spacing`, `Justify`, `PrimaryAxisAlignment`, `CounterAxisAlignment` | Broken layout without explicit alignment |
| 4 | SafeArea: ALWAYS use on mobile pages with TextBox — keyboard WILL obscure inputs without it | Invisible input fields on iOS/Android |
| 5 | TabBar: ALWAYS apply a style (`BottomTabBarStyle`, `TopTabBarStyle`, `VerticalTabBarStyle`) to the TabBar container | Unstyled/invisible TabBar |
| 6 | Card: use `Card` for predefined slots, `CardContentControl` for custom layouts | Wrong control = fighting the template |
| 7 | CommandExtensions: attach `ICommand` to any control event without code-behind | Avoids code-behind event handlers |
| 8 | InputExtensions: `AutoFocusNext` + `ReturnType` for form UX | Poor mobile keyboard experience |
| 9 | ItemsRepeaterExtensions: adds selection + incremental loading to `ItemsRepeater` | Missing selection support on repeater |
| 10 | ShadowContainer: set `Background` on `ShadowContainer`, NOT child — required for inner shadows | Inner shadows silently ignored |

---

## Key Concepts

### Control Categories

| Category | Controls | Purpose |
|----------|----------|---------|
| **Layout** | AutoLayout, SafeArea, Divider | Figma-like flow, safe insets, visual separators |
| **Container** | Card, CardContentControl, ShadowContainer, ZoomContentControl | Content cards, shadows, zoomable areas |
| **Navigation** | TabBar, TabBarItem, NavigationBar | Tab navigation, app bars |
| **Interaction** | Chip, ChipGroup, DrawerControl, DrawerFlyoutPresenter | Selection chips, slide-out drawers |
| **Styling** | ResponsiveExtension, ResponsiveView, StatusBarExtensions | Breakpoint-aware UI, status bar theming |
| **Data Binding** | CommandExtensions, InputExtensions, ItemsRepeaterExtensions | Event-to-command, form input, selection/loading |

### XAML Namespace

Toolkit controls and extensions share one namespace in **page XAML**:

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

The **theme declaration** in App.xaml uses a theme-specific sub-namespace:

| Theme | Namespace |
|-------|-----------|
| Material + Toolkit | `xmlns:utu="using:Uno.Toolkit.UI.Material"` → `MaterialToolkitTheme` |
| Cupertino + Toolkit | `xmlns:utu="using:Uno.Toolkit.UI.Cupertino"` → `CupertinoToolkitTheme` |

---

## Critical Patterns

### AutoLayout vs StackPanel

```xml
<!-- CORRECT: AutoLayout with Figma-like spacing and alignment -->
<utu:AutoLayout Orientation="Vertical" Spacing="16"
                PrimaryAxisAlignment="Center"
                CounterAxisAlignment="Stretch"
                Padding="24">
    <TextBlock Text="Title" Style="{StaticResource TitleLarge}" />
    <TextBox Header="Name" />
    <Button Content="Submit" Style="{StaticResource FilledButtonStyle}" />
</utu:AutoLayout>
```

```xml
<!-- WRONG: StackPanel does not support Spacing, alignment, or Justify -->
<StackPanel Orientation="Vertical">
    <TextBlock Text="Title" Margin="0,0,0,16" />
    <TextBox Header="Name" Margin="0,0,0,16" />
    <Button Content="Submit" />
</StackPanel>
```

### SafeArea for Mobile Forms

```xml
<!-- CORRECT: SafeArea wraps content that needs keyboard avoidance -->
<utu:SafeArea Insets="VisibleBounds,SoftInput" Mode="Padding">
    <ScrollViewer>
        <utu:AutoLayout Orientation="Vertical" Spacing="12" Padding="16">
            <TextBox Header="Email" />
            <TextBox Header="Password" />
            <Button Content="Sign In" />
        </utu:AutoLayout>
    </ScrollViewer>
</utu:SafeArea>
```

```xml
<!-- WRONG: No SafeArea — on-screen keyboard covers TextBox on mobile -->
<ScrollViewer>
    <StackPanel>
        <TextBox Header="Email" />
        <TextBox Header="Password" />
        <Button Content="Sign In" />
    </StackPanel>
</ScrollViewer>
```

### CommandExtensions on TextBox

```xml
<!-- CORRECT: execute search on Enter without code-behind -->
<TextBox Header="Search"
         utu:CommandExtensions.Command="{Binding SearchCommand}" />
```

```xml
<!-- WRONG: requires code-behind KeyDown handler -->
<TextBox Header="Search" KeyDown="SearchBox_KeyDown" />
```

### TabBar Must Have a Style

```xml
<!-- CORRECT: explicit style on TabBar -->
<utu:TabBar Style="{StaticResource BottomTabBarStyle}">
    <utu:TabBarItem Content="Home">
        <utu:TabBarItem.Icon>
            <SymbolIcon Symbol="Home" />
        </utu:TabBarItem.Icon>
    </utu:TabBarItem>
    <utu:TabBarItem Content="Settings">
        <utu:TabBarItem.Icon>
            <SymbolIcon Symbol="Setting" />
        </utu:TabBarItem.Icon>
    </utu:TabBarItem>
</utu:TabBar>
```

```xml
<!-- WRONG: no style — TabBar renders unstyled or invisible -->
<utu:TabBar>
    <utu:TabBarItem Content="Home" />
</utu:TabBar>
```

### ShadowContainer Background Rule

```xml
<!-- CORRECT: Background on the ShadowContainer itself -->
<utu:ShadowContainer Background="White">
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow OffsetX="0" OffsetY="4" BlurRadius="8"
                        Opacity="0.25" Color="Black" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Border CornerRadius="12" Padding="16">
        <TextBlock Text="Card content" />
    </Border>
</utu:ShadowContainer>
```

```xml
<!-- WRONG: Background on child — inner shadows will not render -->
<utu:ShadowContainer>
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow OffsetX="0" OffsetY="4" BlurRadius="8"
                        Opacity="0.25" Color="Black" IsInner="True" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Border Background="White" CornerRadius="12" Padding="16">
        <TextBlock Text="Card content" />
    </Border>
</utu:ShadowContainer>
```

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Using `MaterialTheme` + `ToolkitResources` separately | Duplicate resource keys, unpredictable styles | Use single `MaterialToolkitTheme` (or `CupertinoToolkitTheme`) |
| Missing `Toolkit` in `<UnoFeatures>` | Build error: Toolkit types not found | Add `Toolkit` (or `Material;Toolkit`) to `<UnoFeatures>` — these are separate feature values |
| TabBar without a style (`BottomTabBarStyle`, etc.) | TabBar invisible or unstyled | Always set a `Style` on `TabBar` |
| SafeArea missing on mobile form pages | Keyboard covers TextBox inputs | Wrap form content in `SafeArea` with `Insets="VisibleBounds,SoftInput"` |
| `Background` on child instead of `ShadowContainer` | Inner shadows silently ignored | Set `Background` on `ShadowContainer`, not the child |
| Using `Uno.Toolkit` namespace instead of `Uno.Toolkit.UI` | XAML parse error | Use `xmlns:utu="using:Uno.Toolkit.UI"` |
| AutoLayout without `PrimaryAxisAlignment` or `CounterAxisAlignment` | Children bunched at start with no stretch | Set explicit alignment values |
| Chip without a style (`AssistChipStyle`, etc.) | Chip renders unstyled | Apply the appropriate Chip style |

---

## Related Skills

| Skill | Use When |
|-------|----------|
| `uno-material` | Configuring Material theme, colors, typography, button/TextBox styles |
| `uno-navigation` | Setting up region-based navigation, route registration, NavigationView |
| `mvux` | Building reactive models with IFeed/IState, FeedView, commands |
| `winui-xaml` | General XAML layout, binding, rendering, accessibility |
| `uno-csharp-markup` | Building UI in C# instead of XAML with fluent API |
| `uno-extensions-services` | Hosting, DI, authentication, HTTP, configuration |

---

## Detailed References

| # | File | Covers |
|---|------|--------|
| 1 | [references/01-installation-setup.md](references/01-installation-setup.md) | UnoFeatures, CLI scaffolding, MaterialToolkitTheme vs CupertinoToolkitTheme, C# Markup setup, NuGet packages |
| 2 | [references/02-layout-controls.md](references/02-layout-controls.md) | AutoLayout, SafeArea, Divider, ResponsiveExtension, ResponsiveView |
| 3 | [references/03-container-controls.md](references/03-container-controls.md) | Card, CardContentControl, Chip, ChipGroup, ShadowContainer, ZoomContentControl, DrawerControl, DrawerFlyoutPresenter |
| 4 | [references/04-navigation-controls.md](references/04-navigation-controls.md) | NavigationBar (Windows vs Native mode, MainCommandMode), TabBar, TabBarItem, SelectionIndicator, SegmentedControl, ExtendedSplashScreen, LoadingView |
| 5 | [references/05-interaction-extensions.md](references/05-interaction-extensions.md) | CommandExtensions, InputExtensions, SelectorExtensions, FlipViewExtensions, ItemsRepeaterExtensions |
| 6 | [references/06-styling-theming.md](references/06-styling-theming.md) | Toolkit lightweight styling, the three override scopes, ThemeDictionaries keys, ResourceExtensions.Resources, StatusBarExtensions (iOS/Android), VisualStateManagerExtensions, SystemThemeHelper |
| 7 | [references/07-data-binding-extensions.md](references/07-data-binding-extensions.md) | AncestorBinding, ItemsControlBinding, choosing between them, ProgressExtensions |

---

## Docs Fallback

If the encoded knowledge above may be outdated, use MCP to fetch the latest:

```
uno_platform_docs_search("Uno Toolkit getting started setup")
uno_platform_docs_search("Uno Toolkit AutoLayout orientation spacing")
uno_platform_docs_search("Uno Toolkit SafeArea insets keyboard")
uno_platform_docs_search("Uno Toolkit Card CardContentControl")
uno_platform_docs_search("Uno Toolkit Chip ChipGroup selection")
uno_platform_docs_search("Uno Toolkit TabBar BottomTabBarStyle")
uno_platform_docs_search("Uno Toolkit CommandExtensions")
uno_platform_docs_search("Uno Toolkit InputExtensions AutoFocusNext")
uno_platform_docs_search("Uno Toolkit ShadowContainer inner shadow")
uno_platform_docs_search("Uno Toolkit DrawerControl DrawerFlyoutPresenter")
```
