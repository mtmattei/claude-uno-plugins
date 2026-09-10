---
name: uno-navigation
description: "Uno Platform Navigation Extensions: region-based navigation, route registration, programmatic and XAML navigation, data passing, qualifiers, NavigationView, TabBar, responsive shells, dialogs, and troubleshooting. Use when: (1) Setting up navigation in an Uno Platform app, (2) Defining routes with ViewMap/DataViewMap/RouteMap, (3) Implementing NavigationView or TabBar navigation, (4) Passing data between pages, (5) Showing dialogs or flyouts via navigation, (6) Debugging blank pages or broken navigation. Do NOT use for: MVUX state management (see mvux), general XAML layout (see winui-xaml), new project scaffolding (see uno-platform-agent)."
license: "MIT"
metadata:
  version: "1.0.0"
  author: "Uno Platform"
  category: "uno-platform-navigation"
  tags: [navigation, regions, routing, tabbar, navigationview, uno-platform]
---

# Uno Platform Navigation Extensions

Region-based navigation system for Uno Platform apps. Navigation controls (NavigationView, TabBar) are linked to content areas via regions. Routes map views to ViewModels. Navigation can be triggered from XAML declaratively or from code programmatically.

## Quick Reference

1. Add `Navigation;Toolkit` to `<UnoFeatures>` and call `.UseToolkitNavigation()` in App.xaml.cs
2. Register ALL views with `ViewMap`/`DataViewMap` in `RegisterRoutes` — missing maps cause reflection fallback warnings
3. NEVER put `Region.Attached="True"` inside Shell.xaml or ExtendedSplashScreen — the host isn't ready yet
4. NavigationView MUST use Frame (not Visibility navigator) for its content — Visibility inside NavigationView renders blank
5. TabBar/NavigationView items use `uen:Region.Name` to link to content areas with matching names
6. The root Grid containing both the navigation control and content MUST have `uen:Region.Attached="True"`
7. Qualifiers: `-` = back, `-/` = clear back stack, `!` = dialog; XAML prefix or C# `Qualifiers.*`
8. Data is injected into target ViewModel via constructor parameter — requires `DataViewMap` registration
9. Use `./` prefix in `Navigation.Request` for nested region navigation (ContentControl, Panel)
10. Set `IsDefault: true` on one nested route to auto-navigate on load — missing this causes blank content

## Key Concepts

### Navigation Control Categories

| Category | Control | Behavior |
|---|---|---|
| **Stack-based** | Frame | Forward/backward with back stack |
| **Visibility** | Grid/Panel | Toggles child Visibility (no back stack) |
| **Content** | ContentControl | Replaces Content (no back stack) |
| **Selection** | NavigationView, TabBar | Selects item by Region.Name |
| **Prompt** | ContentDialog, MessageDialog, Flyout, Popup | Opens/closes via dialog qualifier |

### Region System

Regions link navigation controls to content areas using three attached properties:

```xml
xmlns:uen="using:Uno.Extensions.Navigation.UI"

<!-- Region.Attached: enables region-based navigation on a container -->
<Grid uen:Region.Attached="True">

    <!-- Region.Navigator: defines how content is displayed -->
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">

        <!-- Region.Name: identifies which route this area represents -->
        <Grid uen:Region.Name="Home" Visibility="Collapsed">
            <TextBlock Text="Home" />
        </Grid>
    </Grid>

    <utu:TabBar uen:Region.Attached="True">
        <utu:TabBarItem uen:Region.Name="Home" Content="Home" />
    </utu:TabBar>
</Grid>
```

### Route Registration

```csharp
private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        new ViewMap(ViewModel: typeof(ShellViewModel)),
        new ViewMap<MainPage, MainViewModel>(),
        new ViewMap<ProductsPage, ProductsViewModel>(),
        new DataViewMap<DetailPage, DetailViewModel, Product>()
    );

    routes.Register(
        new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
            Nested:
            [
                new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true,
                    Nested:
                    [
                        new RouteMap("Products", View: views.FindByViewModel<ProductsViewModel>(), IsDefault: true),
                        new RouteMap("Detail", View: views.FindByViewModel<DetailViewModel>())
                    ])
            ])
    );
}
```

## Critical Patterns

### NavigationView with Frame (Correct Shell Pattern)

```xml
<!-- CORRECT: Frame inside NavigationView for page navigation -->
<NavigationView uen:Region.Attached="True">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Products" uen:Region.Name="Products" />
        <NavigationViewItem Content="Settings" uen:Region.Name="Settings" />
    </NavigationView.MenuItems>
    <Frame uen:Region.Attached="True" />
</NavigationView>
```

```xml
<!-- WRONG: Visibility navigator inside NavigationView — renders blank -->
<NavigationView uen:Region.Attached="True">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Products" uen:Region.Name="Products" />
    </NavigationView.MenuItems>
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Products" Visibility="Collapsed" />
    </Grid>
</NavigationView>
```

### TabBar with Visibility Navigator

```xml
<!-- CORRECT: TabBar uses Visibility navigator for lightweight content switching -->
<Grid uen:Region.Attached="True">
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home" Visibility="Collapsed">
            <TextBlock Text="Home" />
        </Grid>
        <Grid uen:Region.Name="Search" Visibility="Collapsed">
            <TextBlock Text="Search" />
        </Grid>
    </Grid>
    <utu:TabBar uen:Region.Attached="True">
        <utu:TabBarItem uen:Region.Name="Home" />
        <utu:TabBarItem uen:Region.Name="Search" />
    </utu:TabBar>
</Grid>
```

### XAML Declarative Navigation

```xml
<!-- Navigate forward -->
<Button Content="Details" uen:Navigation.Request="Detail" />

<!-- Navigate back -->
<Button Content="Back" uen:Navigation.Request="-" />

<!-- Clear back stack -->
<Button Content="Logout" uen:Navigation.Request="-/Login" />

<!-- Open dialog -->
<Button Content="Filter" uen:Navigation.Request="!Filter" />

<!-- ListView: selected item auto-passed as data -->
<ListView ItemsSource="{Binding Products}" uen:Navigation.Request="Detail" />
```

### Programmatic Navigation

```csharp
// Get navigator
var nav = this.Navigator(); // from code-behind
// Or inject: public MyViewModel(INavigator navigator)

// By ViewModel type
await nav.NavigateViewModelAsync<DetailViewModel>(this);

// By route name
await nav.NavigateRouteAsync(this, "Detail");

// With data
await nav.NavigateDataAsync(this, myProduct);

// Back
await nav.NavigateBackAsync(this);

// Back with result
await nav.NavigateBackWithResultAsync(this, data: selectedItem);

// Clear back stack
await nav.NavigateViewModelAsync<HomeViewModel>(this, qualifier: Qualifiers.ClearBackStack);
```

## Common Mistakes

**Setup:**
- Missing `UseToolkitNavigation()` in App.xaml.cs — TabBar/NavigationView regions silently fail
- Missing `Navigation;Toolkit` in `<UnoFeatures>` — navigation extensions not available
- `Region.Attached="True"` inside Shell.xaml/ExtendedSplashScreen — host not ready, causes initialization errors
- NavigationView/TabBar defined in ShellView instead of MainPage — regions not functional

**Regions:**
- Root Grid missing `uen:Region.Attached="True"` — TabBar/NavigationView items don't switch content
- `Region.Name` mismatch between navigation item and content area — content doesn't display
- Content regions not initially set to `Visibility="Collapsed"` — all content visible at once
- Using Visibility navigator inside NavigationView — renders blank (must use Frame)

**Routes:**
- Missing `ViewMap`/`DataViewMap` registration — output window shows reflection fallback warning
- Missing `IsDefault: true` on a nested route — app launches to blank content area
- Route names conflicting with control names (List, Grid, Page, Content, Navigation) — resolves to wrong type
- Tab/menu routes not nested under parent page route — entire page replaces instead of just content area
- Route name using suffixes like Page, Model, ViewModel, Service — confuses the route resolver

**Data passing:**
- ViewModel doesn't accept data type as constructor parameter — data silently dropped
- Missing `DataViewMap` registration for the data type — navigation doesn't know which view to show
- Using `Navigation.Data` without `Navigation.Request` — data not sent

**Navigation:**
- Missing `./` prefix for ContentControl/Panel nested navigation — navigates at wrong level
- Forgetting `await` on navigation calls — race conditions or swallowed errors

## Related Skills

| Skill | Use instead when... |
|---|---|
| `mvux` | Implementing reactive data flow with feeds/states (not navigation) |
| `uno-platform-agent` | Creating a new Uno Platform project from scratch |
| `wpf-to-uno-migration` | Migrating WPF navigation patterns (Windows/dialogs → Pages/ContentDialogs) |
| `uno-toolkit` | Using TabBar/NavigationBar control features beyond navigation regions |
| `winui-xaml` | General XAML layout and binding (not navigation-specific) |
| `uno-extensions-services` | Configuring DI, auth, HTTP (not navigation-specific) |

## Detailed References

Read the reference file matching your current task:

- [references/01-setup-host-configuration.md](references/01-setup-host-configuration.md) — Read when setting up navigation in a new project, configuring the host builder, or understanding the region system
- [references/02-routes-viewmaps.md](references/02-routes-viewmaps.md) — Read when defining ViewMap, DataViewMap, ResultDataViewMap, RouteMap, or configuring nested routes
- [references/03-programmatic-navigation.md](references/03-programmatic-navigation.md) — Read when navigating from code-behind or ViewModels, or using XAML declarative navigation
- [references/04-data-and-results.md](references/04-data-and-results.md) — Read when passing data between pages, implementing round-trip data, or using qualifiers
- [references/05-navigation-controls.md](references/05-navigation-controls.md) — Read when implementing NavigationView, TabBar, ContentControl, or Panel/Visibility navigation
- [references/06-responsive-shells.md](references/06-responsive-shells.md) — Read when building adaptive shells that switch between TabBar (mobile) and NavigationView (desktop)
- [references/07-dialogs-flyouts.md](references/07-dialogs-flyouts.md) — Read when showing dialogs, flyouts, message dialogs, or ContentDialogs via navigation

## Docs Fallback

If the encoded knowledge above may be outdated, use MCP to fetch the latest:

```
uno_platform_docs_search("Uno Navigation overview regions setup")
uno_platform_docs_search("Uno Navigation routes ViewMap DataViewMap RouteMap")
uno_platform_docs_search("Uno Navigation INavigator programmatic code")
uno_platform_docs_search("Uno Navigation XAML attached properties Request Data")
uno_platform_docs_search("Uno Navigation qualifiers ClearBackStack dialog")
uno_platform_docs_search("Uno NavigationView region TabBar region")
uno_platform_docs_search("Uno Navigation responsive shell TabBar NavigationView")
uno_platform_docs_search("Uno Navigation dialog flyout ContentDialog MessageDialog")
```
