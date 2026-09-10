# Routes and ViewMaps

How to register views, ViewModels, and data types with ViewMap/DataViewMap/ResultDataViewMap, define hierarchical routes with RouteMap, and configure route naming, defaults, and deep linking.

---

## ViewMap: Associating Views and ViewModels

**Impact**: CRITICAL — Every navigable view MUST be registered. Missing registrations cause reflection fallback warnings and unpredictable behavior.

### View Only

Register a View with no ViewModel (e.g., a static page or a shell):

```csharp
// CORRECT: View-only registration
new ViewMap<SettingsPage>()

// CORRECT: ViewModel-only registration (view resolved by convention)
new ViewMap(ViewModel: typeof(ShellViewModel))
```

### View + ViewModel

The most common pattern. Associates a Page with its ViewModel:

```csharp
// CORRECT: Explicit View + ViewModel association
new ViewMap<MainPage, MainViewModel>()
new ViewMap<ProductsPage, ProductsViewModel>()
new ViewMap<DetailPage, DetailViewModel>()
```

```csharp
// WRONG: Registering View and ViewModel as two separate ViewMaps
new ViewMap<MainPage>(),
new ViewMap(ViewModel: typeof(MainViewModel))
// These are two unlinked registrations — the framework won't associate them
```

### ViewModel Only

When the View is resolved by naming convention (e.g., `MainViewModel` resolves to `MainPage`):

```csharp
// CORRECT: ViewModel-only — framework resolves the View by convention
new ViewMap(ViewModel: typeof(ShellViewModel))
```

This is commonly used for the root Shell, where the View is the navigation host itself.

---

## DataViewMap: Data-Based Navigation

**Impact**: HIGH — Required when navigating with data. Without DataViewMap, passed data is silently dropped.

DataViewMap associates a data type with a View/ViewModel. When you call `NavigateDataAsync<T>()`, the framework finds the DataViewMap registered for type `T` and navigates to the corresponding View.

```csharp
// CORRECT: Associates Product data type with DetailPage/DetailViewModel
new DataViewMap<DetailPage, DetailViewModel, Product>()
```

The data type (`Product`) is injected into the target ViewModel via constructor parameter:

```csharp
// CORRECT: ViewModel receives the navigated data via constructor injection
public class DetailViewModel
{
    public DetailViewModel(Product product)
    {
        CurrentProduct = product;
    }

    public Product CurrentProduct { get; }
}
```

```csharp
// WRONG: ViewModel does not accept data type — data silently dropped
public class DetailViewModel
{
    public DetailViewModel() { }
    // Product data was navigated but never received
}
```

### Polymorphic Data-Based Routing

Different data types auto-route to different pages:

```csharp
views.Register(
    new DataViewMap<ArticlePage, ArticleViewModel, Article>(),
    new DataViewMap<VideoPage, VideoViewModel, Video>(),
    new DataViewMap<PodcastPage, PodcastViewModel, Podcast>()
);

// At runtime: NavigateDataAsync(this, myArticle) goes to ArticlePage
// At runtime: NavigateDataAsync(this, myVideo) goes to VideoPage
```

---

## ResultDataViewMap: Round-Trip Data

**Impact**: HIGH — Required for the `GetDataAsync` pattern where you navigate to a page and await a result.

```csharp
// CORRECT: ResultDataViewMap defines both the input data type and result type
new ResultDataViewMap<FilterPage, FilterViewModel, FilterCriteria, FilterResult>()
//                     View        ViewModel       InputData         ResultData
```

Usage in the calling ViewModel:

```csharp
// Navigate and await result
var result = await navigator.GetDataAsync<FilterResult>(this, data: currentFilter);
if (result is not null)
{
    // Use the returned filter result
}
```

The target ViewModel returns data via `NavigateBackWithResultAsync`:

```csharp
public class FilterViewModel
{
    private readonly INavigator _navigator;
    private readonly FilterCriteria _criteria;

    public FilterViewModel(INavigator navigator, FilterCriteria criteria)
    {
        _navigator = navigator;
        _criteria = criteria;
    }

    public async Task Apply()
    {
        var result = new FilterResult(/* ... */);
        await _navigator.NavigateBackWithResultAsync(this, data: result);
    }
}
```

---

## RouteMap: Named Routes and Hierarchies

**Impact**: CRITICAL — RouteMap defines the navigation structure. Incorrect nesting causes pages to replace entirely instead of loading in content areas.

### Basic RouteMap

```csharp
routes.Register(
    new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
        Nested:
        [
            new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true)
        ])
);
```

### RouteMap Parameters

| Parameter | Type | Purpose |
|---|---|---|
| `Path` (1st arg) | `string` | Route name used in navigation requests |
| `View` | `ViewMap` | The ViewMap to display for this route |
| `Nested` | `RouteMap[]` | Child routes accessible from this route's content |
| `IsDefault` | `bool` | Auto-navigate to this route when parent loads |
| `DependsOn` | `string` | Route that must be loaded first |
| `Init` | `Func<...>` | Initialization logic run before the route loads |

### Hierarchical Nesting

```csharp
// CORRECT: Full route hierarchy — tabs are nested under Main
routes.Register(
    new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
        Nested:
        [
            new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true,
                Nested:
                [
                    new RouteMap("Home", View: views.FindByViewModel<HomeViewModel>(), IsDefault: true),
                    new RouteMap("Products", View: views.FindByViewModel<ProductsViewModel>(),
                        Nested:
                        [
                            new RouteMap("Detail", View: views.FindByViewModel<DetailViewModel>())
                        ]),
                    new RouteMap("Settings", View: views.FindByViewModel<SettingsViewModel>())
                ])
        ])
);
```

```csharp
// WRONG: Tab routes not nested under Main — navigating to "Products"
// replaces the entire page instead of just the content area
routes.Register(
    new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
        Nested:
        [
            new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true),
            new RouteMap("Products", View: views.FindByViewModel<ProductsViewModel>()),
            new RouteMap("Settings", View: views.FindByViewModel<SettingsViewModel>())
        ])
);
```

### IsDefault Is Required for Initial Content

**Impact**: HIGH — Missing `IsDefault: true` on a nested route causes the app to launch with a blank content area.

```csharp
// CORRECT: One child route marked as default
new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true,
    Nested:
    [
        new RouteMap("Home", View: views.FindByViewModel<HomeViewModel>(), IsDefault: true),
        new RouteMap("Search", View: views.FindByViewModel<SearchViewModel>())
    ])
```

```csharp
// WRONG: No default — content area is blank on load
new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true,
    Nested:
    [
        new RouteMap("Home", View: views.FindByViewModel<HomeViewModel>()),
        new RouteMap("Search", View: views.FindByViewModel<SearchViewModel>())
    ])
```

---

## View Lookups: FindByViewModel and FindByView

**Impact**: MEDIUM — Use these to reference ViewMaps in RouteMap registration instead of repeating type information.

```csharp
// CORRECT: Look up the ViewMap by ViewModel type
new RouteMap("Main", View: views.FindByViewModel<MainViewModel>())

// CORRECT: Look up the ViewMap by View type
new RouteMap("Main", View: views.FindByView<MainPage>())
```

These methods search the `IViewRegistry` for the matching registration. The ViewModel or View must already be registered via `views.Register(...)` before it can be looked up.

---

## Route Naming Rules

**Impact**: HIGH — Bad route names cause the framework to resolve to the wrong type or fail silently.

### Names to Avoid

Do NOT use names that match common control type names or their suffixes:

```csharp
// WRONG: "List", "Grid", "Page", "Content", "Navigation" are control type names
new RouteMap("List", ...)        // Resolves to ListView/ListBox
new RouteMap("Grid", ...)        // Resolves to Grid control
new RouteMap("Page", ...)        // Resolves to Page base type
new RouteMap("Content", ...)     // Resolves to ContentControl
new RouteMap("Navigation", ...)  // Resolves to NavigationView
```

```csharp
// WRONG: Suffixes like Page, Model, ViewModel, Service confuse the resolver
new RouteMap("ProductsPage", ...)
new RouteMap("ProductsViewModel", ...)
new RouteMap("ProductsModel", ...)
new RouteMap("ProductsService", ...)
```

```csharp
// CORRECT: Use clean domain names
new RouteMap("Products", ...)
new RouteMap("Detail", ...)
new RouteMap("Settings", ...)
new RouteMap("Profile", ...)
```

---

## FromQuery / ToQuery for Deep Linking

**Impact**: MEDIUM — Enables URL-based deep linking and state restoration.

Implement `FromQuery` and `ToQuery` on data types to support deep link URLs:

```csharp
public record ProductFilter(string Category, int MinPrice)
{
    // Serialize to query string parameters
    public static ProductFilter FromQuery(IDictionary<string, string> query)
    {
        return new ProductFilter(
            query.GetValueOrDefault("category", "All"),
            int.TryParse(query.GetValueOrDefault("minPrice"), out var price) ? price : 0
        );
    }

    public IDictionary<string, string> ToQuery()
    {
        return new Dictionary<string, string>
        {
            ["category"] = Category,
            ["minPrice"] = MinPrice.ToString()
        };
    }
}
```

---

## Complete RegisterRoutes Example

**Impact**: HIGH — Reference pattern for a typical multi-tab app with detail navigation.

```csharp
private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        // Shell and main layout
        new ViewMap(ViewModel: typeof(ShellViewModel)),
        new ViewMap<MainPage, MainViewModel>(),

        // Tab pages
        new ViewMap<HomePage, HomeViewModel>(),
        new ViewMap<ProductsPage, ProductsViewModel>(),
        new ViewMap<SettingsPage, SettingsViewModel>(),

        // Detail pages with data
        new DataViewMap<ProductDetailPage, ProductDetailViewModel, Product>(),

        // Round-trip pages
        new ResultDataViewMap<FilterPage, FilterViewModel, FilterCriteria, FilterResult>()
    );

    routes.Register(
        new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
            Nested:
            [
                new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(), IsDefault: true,
                    Nested:
                    [
                        new RouteMap("Home", View: views.FindByViewModel<HomeViewModel>(), IsDefault: true),
                        new RouteMap("Products", View: views.FindByViewModel<ProductsViewModel>(),
                            Nested:
                            [
                                new RouteMap("ProductDetail", View: views.FindByViewModel<ProductDetailViewModel>())
                            ]),
                        new RouteMap("Settings", View: views.FindByViewModel<SettingsViewModel>())
                    ]),
                new RouteMap("Filter", View: views.FindByViewModel<FilterViewModel>())
            ])
    );
}
```

---

## Reference

- [Define Routes Walkthrough](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/DefineRoutes.html)
- [Register Routes](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/RegisterRoutes.html)
- [Navigation Overview](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/NavigationOverview.html)
- [Passing Navigation Data (Chefs)](https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/PassingNavigationData.html)
