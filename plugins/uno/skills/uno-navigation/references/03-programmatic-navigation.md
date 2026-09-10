# Programmatic and XAML Navigation

How to trigger navigation from code-behind, ViewModels, and XAML using INavigator methods and attached properties.

---

## Getting INavigator

**Impact**: CRITICAL — INavigator is the gateway to all programmatic navigation. Getting it wrong means no navigation occurs.

### From Code-Behind

```csharp
// CORRECT: Extension method on any FrameworkElement
var nav = this.Navigator();
await nav.NavigateViewModelAsync<DetailViewModel>(this);
```

### From ViewModel via Constructor Injection

```csharp
// CORRECT: INavigator injected by DI container
public class ProductsViewModel
{
    private readonly INavigator _navigator;

    public ProductsViewModel(INavigator navigator)
    {
        _navigator = navigator;
    }

    public async Task GoToDetail(Product product)
    {
        await _navigator.NavigateDataAsync(this, product);
    }
}
```

```csharp
// WRONG: Trying to resolve INavigator manually — use DI injection
public class ProductsViewModel
{
    public ProductsViewModel()
    {
        // No way to get INavigator without injection
    }
}
```

---

## Navigation Methods

**Impact**: HIGH — Choose the right method for your scenario. Using the wrong one leads to failed navigation or missing data.

### NavigateViewModelAsync — Navigate by ViewModel Type

The most common method. Routes to the page registered for the specified ViewModel:

```csharp
// CORRECT: Navigate to the route associated with DetailViewModel
await navigator.NavigateViewModelAsync<DetailViewModel>(this);
```

### NavigateRouteAsync — Navigate by Route Name

Navigate using the string route name defined in RouteMap:

```csharp
// CORRECT: Navigate to the route named "Detail"
await navigator.NavigateRouteAsync(this, "Detail");
```

### NavigateViewAsync — Navigate by View Type

Navigate directly to a specific View:

```csharp
// CORRECT: Navigate to DetailPage directly
await navigator.NavigateViewAsync<DetailPage>(this);
```

### NavigateDataAsync — Navigate with Data

Navigate and pass a data object. Requires `DataViewMap` registration:

```csharp
// CORRECT: Navigate with data — framework resolves the target from data type
await navigator.NavigateDataAsync(this, selectedProduct);
```

### NavigateBackAsync — Go Back

```csharp
// CORRECT: Navigate back one level in the back stack
await navigator.NavigateBackAsync(this);
```

### NavigateBackWithResultAsync — Go Back with Data

Returns data to the previous page (used with `ResultDataViewMap`):

```csharp
// CORRECT: Return result to the caller
await navigator.NavigateBackWithResultAsync(this, data: selectedFilter);
```

### ForResult Variants — Await a Navigation Result

Navigate to a page and await the result when the user navigates back:

```csharp
// CORRECT: Navigate and await the result
var result = await navigator.NavigateRouteForResultAsync<FilterResult>(this, "Filter");

// CORRECT: GetDataAsync shorthand
var result = await navigator.GetDataAsync<FilterResult>(this, data: currentFilter);
if (result is not null)
{
    ApplyFilter(result);
}
```

---

## Multi-Page Navigation (Back Stack Building)

**Impact**: MEDIUM — Slash-separated routes create intermediate back stack entries.

```csharp
// CORRECT: "Products/Detail" navigates to Products, then to Detail
// Creates a back stack: [Products, Detail]
// Pressing back from Detail returns to Products
await navigator.NavigateRouteAsync(this, "Products/Detail");
```

```csharp
// WRONG: Navigating directly to "Detail" skips Products in the back stack
await navigator.NavigateRouteAsync(this, "Detail");
// Back from Detail goes to wherever you were before, not Products
```

---

## Always Await Navigation Calls

**Impact**: HIGH — Forgetting `await` causes race conditions or swallowed errors.

```csharp
// CORRECT: await the navigation call
await navigator.NavigateViewModelAsync<DetailViewModel>(this);

// WRONG: fire-and-forget — race condition, errors swallowed
navigator.NavigateViewModelAsync<DetailViewModel>(this);
// No await!
```

---

## XAML Declarative Navigation

**Impact**: HIGH — Enables navigation directly from XAML without code-behind.

### XAML Namespace

```xml
xmlns:uen="using:Uno.Extensions.Navigation.UI"
```

### Navigation.Request — Basic Forward Navigation

```xml
<!-- CORRECT: Navigate to "Detail" route on button click -->
<Button Content="View Details" uen:Navigation.Request="Detail" />
```

### Navigation.Data — Pass Data with Navigation

```xml
<!-- CORRECT: Pass bound data with the navigation request -->
<Button Content="View Details"
        uen:Navigation.Request="Detail"
        uen:Navigation.Data="{Binding SelectedProduct}" />
```

```xml
<!-- WRONG: Navigation.Data without Navigation.Request — data not sent -->
<Button Content="View Details"
        uen:Navigation.Data="{Binding SelectedProduct}" />
```

### Auto-Behavior on Interactive Controls

Different controls trigger navigation on different interactions:

| Control | Trigger | Data Auto-Passed |
|---|---|---|
| `Button` | Click | None (use Navigation.Data) |
| `ListView` | SelectionChanged | Selected item |
| `ItemsRepeater` | Item tapped | Tapped item |
| `NavigationViewItem` | Selected | None |

```xml
<!-- CORRECT: ListView auto-passes selected item as data -->
<ListView ItemsSource="{Binding Products}"
          uen:Navigation.Request="Detail">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
<!-- When user selects a Product, it navigates to "Detail" with that Product as data -->
```

### Back Navigation

```xml
<!-- CORRECT: Navigate back -->
<Button Content="Back" uen:Navigation.Request="-" />
```

### Qualifier Prefixes in XAML

```xml
<!-- Clear back stack and navigate (e.g., after login) -->
<Button Content="Continue" uen:Navigation.Request="-/Home" />

<!-- Open as dialog -->
<Button Content="Filter" uen:Navigation.Request="!Filter" />

<!-- Remove current page from back stack and navigate -->
<Button Content="Skip" uen:Navigation.Request="-Detail" />
```

---

## Nested Region Navigation with ./ Prefix

**Impact**: HIGH — Without the `./` prefix, navigation targets the wrong region level.

When navigating within a ContentControl or Panel region (not the main Frame), prefix with `./`:

```xml
<!-- CORRECT: Navigate within nested ContentControl region -->
<Button Content="Show Details"
        uen:Navigation.Request="./DetailContent" />
```

```xml
<!-- WRONG: Without ./ prefix, targets the parent frame instead of nested region -->
<Button Content="Show Details"
        uen:Navigation.Request="DetailContent" />
```

From code:

```csharp
// CORRECT: Navigate within nested region
await navigator.NavigateRouteAsync(this, "./DetailContent");
```

---

## Summary: Choosing the Right Navigation Method

| Scenario | Code Approach | XAML Approach |
|---|---|---|
| Go to a known page | `NavigateViewModelAsync<T>()` | `Navigation.Request="Route"` |
| Go to page by route name | `NavigateRouteAsync("Route")` | `Navigation.Request="Route"` |
| Pass data to target | `NavigateDataAsync(data)` | `Navigation.Data="{Binding}"` |
| Go back | `NavigateBackAsync()` | `Navigation.Request="-"` |
| Return a result | `NavigateBackWithResultAsync(data)` | N/A (code only) |
| Await a result | `GetDataAsync<T>()` | N/A (code only) |
| Clear back stack | `qualifier: Qualifiers.ClearBackStack` | `Navigation.Request="-/Route"` |
| Open dialog | `qualifier: Qualifiers.Dialog` | `Navigation.Request="!Route"` |

---

## Reference

- [Navigation Overview](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/NavigationOverview.html)
- [Navigation via Code Behind (Chefs)](https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/NavigationCodeBehind.html)
- [XAML Attached Properties Walkthrough](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/NavigateUsingAttachedPropertiesInXAML.html)
- [XAML Navigation (Chefs)](https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/XamlNavigation.html)
- [Advanced Page Navigation](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/AdvancedPageNavigation.html)
