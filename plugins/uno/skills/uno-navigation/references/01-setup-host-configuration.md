# Setup and Host Configuration

How to add Navigation Extensions to an Uno Platform project, configure the host builder, declare the Host property, and set up the region system.

---

## Adding Navigation to the Project

**Impact**: CRITICAL — Without the correct UnoFeatures, navigation extensions are not available and all region-based navigation silently fails.

### UnoFeatures in .csproj

```xml
<!-- CORRECT: Both Navigation and Toolkit are required for TabBar/NavigationView regions -->
<UnoFeatures>Navigation;Toolkit</UnoFeatures>
```

```xml
<!-- WRONG: Missing Toolkit — TabBar/NavigationView region support not available -->
<UnoFeatures>Navigation</UnoFeatures>

<!-- WRONG: Missing Navigation entirely — no navigation extensions at all -->
<UnoFeatures>Toolkit</UnoFeatures>
```

- `Navigation` provides core navigation, routing, INavigator, and region system
- `Toolkit` adds TabBar, NavigationBar, and toolkit-specific navigation integration
- Both are required for a typical app with TabBar or NavigationView navigation

---

## Host Builder Configuration in App.xaml.cs

**Impact**: CRITICAL — Missing `UseToolkitNavigation()` causes regions to silently fail with no error output.

### Correct Host Builder Setup

```csharp
// CORRECT: Full navigation host builder configuration
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args)
        .Configure(hostBuilder =>
        {
            hostBuilder
                .UseToolkitNavigation()
                .ConfigureServices(services =>
                {
                    // Register your services here
                })
                .UseNavigation(RegisterRoutes);
        });

    Host = appBuilder.Build();

    // ...
}

private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        new ViewMap(ViewModel: typeof(ShellViewModel)),
        new ViewMap<MainPage, MainViewModel>()
    );

    routes.Register(
        new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
            Nested:
            [
                new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true)
            ])
    );
}
```

```csharp
// WRONG: Missing UseToolkitNavigation — TabBar/NavigationView regions silently broken
hostBuilder
    .UseNavigation(RegisterRoutes);

// WRONG: UseNavigation without RegisterRoutes — no routes registered, blank pages
hostBuilder
    .UseToolkitNavigation()
    .UseNavigation();
```

### The Host Property Declaration

The `Host` property is required on `App.xaml.cs` for the navigation system to function:

```csharp
// CORRECT: App.xaml.cs must expose the Host property
public class App : Application
{
    public IHost? Host { get; private set; }

    // ...
}
```

```csharp
// WRONG: Missing Host property — navigation host cannot be resolved
public class App : Application
{
    // No Host property declared — navigation system has no entry point
}
```

---

## Region System

**Impact**: CRITICAL — Regions are the core mechanism linking navigation controls to content areas. Misconfigured regions produce blank content with no error.

### Attached Properties

Three attached properties control the region system. XAML namespace:

```xml
xmlns:uen="using:Uno.Extensions.Navigation.UI"
```

| Property | Purpose | Example |
|---|---|---|
| `Region.Attached="True"` | Enables region-based navigation on a container | Root Grid, NavigationView, TabBar |
| `Region.Name="RouteName"` | Identifies which route this region represents | Tab items, NavigationView items, content panels |
| `Region.Navigator="Visibility"` | Defines how child content is displayed | Grid/Panel for visibility switching |

### Root Grid Must Have Region.Attached

The outermost Grid containing both the navigation control and its content area MUST have `Region.Attached="True"`:

```xml
<!-- CORRECT: Root Grid has Region.Attached wrapping both nav control and content -->
<Grid uen:Region.Attached="True">
    <Grid.RowDefinitions>
        <RowDefinition Height="*" />
        <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>

    <!-- Content area -->
    <Grid Grid.Row="0"
          uen:Region.Attached="True"
          uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home" Visibility="Collapsed">
            <local:HomePage />
        </Grid>
        <Grid uen:Region.Name="Search" Visibility="Collapsed">
            <local:SearchPage />
        </Grid>
    </Grid>

    <!-- Navigation control -->
    <utu:TabBar Grid.Row="1" uen:Region.Attached="True">
        <utu:TabBarItem uen:Region.Name="Home" Content="Home" />
        <utu:TabBarItem uen:Region.Name="Search" Content="Search" />
    </utu:TabBar>
</Grid>
```

```xml
<!-- WRONG: Root Grid missing Region.Attached — tabs do not switch content -->
<Grid>
    <Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
        <Grid uen:Region.Name="Home" Visibility="Collapsed" />
    </Grid>
    <utu:TabBar uen:Region.Attached="True">
        <utu:TabBarItem uen:Region.Name="Home" Content="Home" />
    </utu:TabBar>
</Grid>
```

### Visibility Navigator

For lightweight content switching without a back stack, use `Region.Navigator="Visibility"`:

```xml
<!-- CORRECT: Visibility navigator toggles child Visibility, children start Collapsed -->
<Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
    <Grid uen:Region.Name="Home" Visibility="Collapsed">
        <TextBlock Text="Home Content" />
    </Grid>
    <Grid uen:Region.Name="Settings" Visibility="Collapsed">
        <TextBlock Text="Settings Content" />
    </Grid>
</Grid>
```

```xml
<!-- WRONG: Children not set to Visibility="Collapsed" — all content visible at once -->
<Grid uen:Region.Attached="True" uen:Region.Navigator="Visibility">
    <Grid uen:Region.Name="Home">
        <TextBlock Text="Home Content" />
    </Grid>
    <Grid uen:Region.Name="Settings">
        <TextBlock Text="Settings Content" />
    </Grid>
</Grid>
```

---

## Critical: Never Put Region.Attached Inside Shell.xaml or ExtendedSplashScreen

**Impact**: CRITICAL — Causes initialization errors because the navigation host is not ready when Shell.xaml or ExtendedSplashScreen content is first loaded.

```xml
<!-- WRONG: Region.Attached inside Shell.xaml content — host not ready -->
<!-- Shell.xaml -->
<Page>
    <Grid uen:Region.Attached="True">
        <utu:TabBar uen:Region.Attached="True">
            <utu:TabBarItem uen:Region.Name="Home" />
        </utu:TabBar>
    </Grid>
</Page>
```

```xml
<!-- CORRECT: Region.Attached belongs in MainPage.xaml (or whichever page
     is loaded AFTER the Shell/ExtendedSplashScreen completes) -->
<!-- MainPage.xaml -->
<Page>
    <Grid uen:Region.Attached="True">
        <utu:TabBar uen:Region.Attached="True">
            <utu:TabBarItem uen:Region.Name="Home" />
        </utu:TabBar>
    </Grid>
</Page>
```

The Shell is the navigation host container — it provides the root frame. Navigation regions must be defined in pages loaded by the navigation system, not in the host itself.

---

## Region.Name Matching Rules

**Impact**: HIGH — Mismatched names between navigation items and content areas cause content to not display.

```xml
<!-- CORRECT: Region.Name values match exactly between TabBarItem and content Grid -->
<utu:TabBarItem uen:Region.Name="Products" Content="Products" />
<!-- ... -->
<Grid uen:Region.Name="Products" Visibility="Collapsed" />
```

```xml
<!-- WRONG: Name mismatch — "Product" vs "Products" -->
<utu:TabBarItem uen:Region.Name="Product" Content="Products" />
<Grid uen:Region.Name="Products" Visibility="Collapsed" />
```

Region names must also match the route names registered in `RegisterRoutes`. If the route is named `"Products"`, the `Region.Name` must be `"Products"`.

---

## Reference

- [Navigation Overview](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/NavigationOverview.html)
- [Define Regions Walkthrough](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/DefineRegions.html)
- [Region How-To](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/HowTo-Regions.html)
- [Navigation Region Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Navigation/NavigationRegion.html)
