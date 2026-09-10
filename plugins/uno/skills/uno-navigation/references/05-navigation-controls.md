# Navigation Controls

How to implement NavigationView, TabBar, ContentControl, and Panel/Grid visibility navigation using Uno Platform's region-based navigation system.

---

## NavigationView + Frame

**Impact**: CRITICAL — The standard desktop/sidebar navigation pattern. Frame is the ONLY valid content host inside NavigationView.

### Basic Setup

NavigationView items use `Region.Name` to identify routes. A Frame inside the NavigationView hosts the actual page content.

```xml
<!-- CORRECT: NavigationView with Frame content host -->
<NavigationView uen:Region.Attached="True">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" uen:Region.Name="Home">
            <NavigationViewItem.Icon>
                <SymbolIcon Symbol="Home" />
            </NavigationViewItem.Icon>
        </NavigationViewItem>
        <NavigationViewItem Content="Products" uen:Region.Name="Products">
            <NavigationViewItem.Icon>
                <SymbolIcon Symbol="Shop" />
            </NavigationViewItem.Icon>
        </NavigationViewItem>
    </NavigationView.MenuItems>
    <Frame uen:Region.Attached="True" />
</NavigationView>
```

```xml
<!-- WRONG: Visibility navigator inside NavigationView — renders blank page -->
<NavigationView uen:Region.Attached="True">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" uen:Region.Name="Home" />
        <NavigationViewItem Content="Products" uen:Region.Name="Products" />
    </NavigationView.MenuItems>
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home" Visibility="Collapsed">
            <TextBlock Text="Home" />
        </Grid>
        <Grid uen:Region.Name="Products" Visibility="Collapsed">
            <TextBlock Text="Products" />
        </Grid>
    </Grid>
</NavigationView>
```

### SettingsItem Region Name

NavigationView's SettingsItem cannot have `Region.Name` set in XAML because it is created internally. Set it in code-behind.

```csharp
// CORRECT: set SettingsItem region name in code-behind
public MainPage()
{
    InitializeComponent();

    // MyNavigationView is x:Name on the NavigationView
    Region.SetName(
        (NavigationViewItem)MyNavigationView.SettingsItem,
        "Settings");
}
```

```csharp
// WRONG: trying to set Region.Name on SettingsItem in XAML — not accessible
// There is no NavigationView.SettingsItem XAML property that accepts attached properties
```

### Nested Routes for Content-Only Updates

Routes for NavigationView items MUST be nested under the parent page route so only the content area updates, not the entire page.

```csharp
// CORRECT: Home and Products nested under Main — only Frame content swaps
routes.Register(
    new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
        Nested:
        [
            new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true,
                Nested:
                [
                    new RouteMap("Home", View: views.FindByViewModel<HomeViewModel>(), IsDefault: true),
                    new RouteMap("Products", View: views.FindByViewModel<ProductsViewModel>()),
                    new RouteMap("Settings", View: views.FindByViewModel<SettingsViewModel>())
                ])
        ])
);
```

```csharp
// WRONG: Home and Products at same level as Main — entire page replaces
routes.Register(
    new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
        Nested:
        [
            new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true),
            new RouteMap("Home", View: views.FindByViewModel<HomeViewModel>()),
            new RouteMap("Products", View: views.FindByViewModel<ProductsViewModel>())
        ])
);
```

---

## TabBar + Visibility Navigator

**Impact**: CRITICAL — The standard mobile/bottom navigation pattern. Requires specific container setup and Toolkit.

### Prerequisites

TabBar navigation requires both `UseToolkitNavigation()` and `Toolkit` in UnoFeatures:

```xml
<!-- In .csproj -->
<UnoFeatures>Navigation;Toolkit</UnoFeatures>
```

```csharp
// In App.xaml.cs host builder
.UseToolkitNavigation()
```

### Basic Setup

The root Grid MUST have `Region.Attached`. Content Grid uses `Region.Navigator="Visibility"`. All content children start `Visibility="Collapsed"`.

```xml
<!-- CORRECT: TabBar with Visibility navigator -->
<Grid uen:Region.Attached="True">

    <!-- Content area: Visibility navigator toggles children -->
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home" Visibility="Collapsed">
            <local:HomePage />
        </Grid>
        <Grid uen:Region.Name="Search" Visibility="Collapsed">
            <local:SearchPage />
        </Grid>
        <Grid uen:Region.Name="Profile" Visibility="Collapsed">
            <local:ProfilePage />
        </Grid>
    </Grid>

    <!-- Navigation control: TabBarItem names match content Region.Name -->
    <utu:TabBar uen:Region.Attached="True"
                Style="{StaticResource MaterialBottomTabBarStyle}">
        <utu:TabBarItem uen:Region.Name="Home" Content="Home"
                        Style="{StaticResource MaterialBottomTabBarItemStyle}">
            <utu:TabBarItem.Icon>
                <SymbolIcon Symbol="Home" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
        <utu:TabBarItem uen:Region.Name="Search" Content="Search"
                        Style="{StaticResource MaterialBottomTabBarItemStyle}">
            <utu:TabBarItem.Icon>
                <SymbolIcon Symbol="Find" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
        <utu:TabBarItem uen:Region.Name="Profile" Content="Profile"
                        Style="{StaticResource MaterialBottomTabBarItemStyle}">
            <utu:TabBarItem.Icon>
                <SymbolIcon Symbol="Contact" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
    </utu:TabBar>

</Grid>
```

```xml
<!-- WRONG: missing Region.Attached on root Grid — tabs don't switch content -->
<Grid>
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home" Visibility="Collapsed" />
    </Grid>
    <utu:TabBar uen:Region.Attached="True">
        <utu:TabBarItem uen:Region.Name="Home" />
    </utu:TabBar>
</Grid>
```

```xml
<!-- WRONG: content not initially Collapsed — all tabs visible at once -->
<Grid uen:Region.Attached="True">
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home">
            <TextBlock Text="Home" />
        </Grid>
        <Grid uen:Region.Name="Search">
            <TextBlock Text="Search" />
        </Grid>
    </Grid>
    ...
</Grid>
```

---

## ContentControl

**Impact**: HIGH — Lightweight content replacement without back stack. Good for master-detail or side panels.

### Basic Setup

ContentControl replaces its `Content` when navigated. Use `./` prefix in `Navigation.Request` to target the specific region.

```xml
<!-- CORRECT: ContentControl region navigation with ./ prefix -->
<Grid uen:Region.Attached="True">
    <Grid>
        <Button Content="Show Left"
                uen:Navigation.Request="./Left" />
        <Button Content="Show Right"
                uen:Navigation.Request="./Right" />
    </Grid>
    <ContentControl uen:Region.Attached="True" />
</Grid>
```

```xml
<!-- WRONG: missing ./ prefix — navigates at parent level instead of ContentControl -->
<Grid uen:Region.Attached="True">
    <Button Content="Show Left"
            uen:Navigation.Request="Left" />
    <ContentControl uen:Region.Attached="True" />
</Grid>
```

### Named ContentControl Regions

Use `Region.Name` to target specific ContentControl regions when you have multiple on the same page.

```xml
<Grid uen:Region.Attached="True">
    <Grid>
        <Button Content="Show Left Panel"
                uen:Navigation.Request="./Details/Left" />
        <Button Content="Show Right Panel"
                uen:Navigation.Request="./Details/Right" />
    </Grid>
    <ContentControl uen:Region.Attached="True"
                    uen:Region.Name="Details" />
</Grid>
```

### Behavior

- No back stack — content is simply replaced
- Previous content is disposed when new content loads
- Great for dynamic panels where history is not needed

---

## Panel/Grid Visibility Navigator

**Impact**: MEDIUM — Zero-overhead content switching that keeps all children in the visual tree.

### Basic Setup

Attach `Region.Navigator="Visibility"` to a container Grid. Children use `Region.Name` and start Collapsed. Navigate with `./` prefix.

```xml
<!-- CORRECT: Grid Visibility navigator -->
<Grid uen:Region.Attached="True">
    <StackPanel>
        <Button Content="Show Alpha"
                uen:Navigation.Request="./Alpha" />
        <Button Content="Show Beta"
                uen:Navigation.Request="./Beta" />
    </StackPanel>

    <Grid uen:Region.Attached="True"
          uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Alpha" Visibility="Collapsed">
            <TextBlock Text="Alpha content" />
        </Grid>
        <Grid uen:Region.Name="Beta" Visibility="Collapsed">
            <TextBlock Text="Beta content" />
        </Grid>
    </Grid>
</Grid>
```

```xml
<!-- WRONG: missing Region.Navigator="Visibility" — no content switching happens -->
<Grid uen:Region.Attached="True">
    <Grid uen:Region.Attached="True">
        <Grid uen:Region.Name="Alpha" Visibility="Collapsed" />
        <Grid uen:Region.Name="Beta" Visibility="Collapsed" />
    </Grid>
</Grid>
```

### When to Use Visibility vs Frame vs ContentControl

| Feature | Visibility (Panel/Grid) | Frame | ContentControl |
|---|---|---|---|
| Back stack | No | Yes | No |
| Content lifetime | Always alive in visual tree | Created/destroyed on navigate | Replaced on navigate |
| Memory | Higher (all children always exist) | Lower (only current page) | Lower (only current content) |
| Switch speed | Instant (toggle Visibility) | Page creation overhead | Content creation overhead |
| Best for | Tabs, toggles, settings panels | Page-based navigation | Dynamic detail panels |

---

## Common Mistakes

- **Visibility navigator inside NavigationView**: Renders blank. NavigationView MUST use Frame as its content host.
- **Missing `Region.Attached="True"` on root Grid**: TabBar/NavigationView items don't switch content.
- **Region.Name mismatch**: Navigation item `Region.Name="home"` won't match content `Region.Name="Home"` — names are case-sensitive.
- **Missing `./` prefix**: Without it, ContentControl/Panel navigation targets the parent level instead of the nested region.
- **Content not initially Collapsed**: In Visibility navigator, all children show at once if not set to `Visibility="Collapsed"`.
- **Routes not nested under parent**: NavigationView/TabBar items replace the entire page instead of just the content area.

---

## Reference

- [Navigation Regions Overview](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Regions.html)
- [NavigationView Region](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/NavigationView.html)
- [TabBar Region](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/TabBar.html)
- [Panel Visibility Navigation](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/PanelVisibility.html)
- [ContentControl Navigation](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/ContentControl.html)
