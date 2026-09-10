# Data Passing, Results, and Qualifiers

How to pass data forward during navigation, receive results back, use data-based routing, and control back stack behavior with qualifiers.

---

## Forward Data: Navigate with Data

**Impact**: CRITICAL — Data passing requires three things aligned: DataViewMap registration, NavigateDataAsync call, and constructor parameter in target ViewModel. Missing any one silently fails.

### Step 1: Register DataViewMap

```csharp
views.Register(
    new DataViewMap<ProductDetailPage, ProductDetailViewModel, Product>()
);
```

### Step 2: Navigate with Data

```csharp
// CORRECT: From code — pass the data object
await navigator.NavigateDataAsync(this, selectedProduct);
```

### Step 3: Receive Data via Constructor Injection

```csharp
// CORRECT: Target ViewModel receives data via constructor
public class ProductDetailViewModel
{
    public Product Product { get; }

    public ProductDetailViewModel(Product product)
    {
        Product = product;
    }
}
```

### Complete Correct Pattern

```csharp
// Registration
new DataViewMap<ProductDetailPage, ProductDetailViewModel, Product>()

// Source ViewModel
public async Task ViewProduct(Product product)
{
    await _navigator.NavigateDataAsync(this, product);
}

// Target ViewModel
public class ProductDetailViewModel
{
    public Product Product { get; }
    public ProductDetailViewModel(Product product) => Product = product;
}
```

### Common Mistakes

```csharp
// WRONG: Missing DataViewMap — using ViewMap instead
new ViewMap<ProductDetailPage, ProductDetailViewModel>()
// NavigateDataAsync works but data is silently dropped

// WRONG: ViewModel has no constructor parameter for the data type
public class ProductDetailViewModel
{
    public ProductDetailViewModel() { }
    // Product data never received
}

// WRONG: Constructor parameter type does not match DataViewMap data type
new DataViewMap<ProductDetailPage, ProductDetailViewModel, Product>()
// ...
public ProductDetailViewModel(int productId) { } // Expects int, not Product
```

---

## Data-Based Routing (Polymorphic)

**Impact**: HIGH — Different data types automatically route to different pages without explicit route names.

```csharp
// Registration: each data type maps to a different view
views.Register(
    new DataViewMap<ArticlePage, ArticleViewModel, Article>(),
    new DataViewMap<VideoPage, VideoViewModel, Video>(),
    new DataViewMap<PodcastPage, PodcastViewModel, Podcast>()
);

// Usage: pass any content item — framework resolves the correct page
public async Task ViewContent(object content)
{
    // If content is Article -> ArticlePage
    // If content is Video -> VideoPage
    // If content is Podcast -> PodcastPage
    await navigator.NavigateDataAsync(this, content);
}
```

This is powerful for feed/list scenarios where items are heterogeneous:

```xml
<!-- CORRECT: ListView with mixed item types, each navigates to its own page -->
<ListView ItemsSource="{Binding ContentItems}"
          uen:Navigation.Request="Detail" />
```

---

## Navigation.Data in XAML

**Impact**: HIGH — Enables data passing directly from XAML without code-behind.

### One-Way Data Passing

```xml
<!-- CORRECT: Pass data from a binding -->
<Button Content="View Details"
        uen:Navigation.Request="ProductDetail"
        uen:Navigation.Data="{Binding SelectedProduct}" />
```

### TwoWay Binding for Round-Trip

When the target page modifies the data and navigates back, TwoWay binding propagates the change:

```xml
<!-- CORRECT: TwoWay binding enables round-trip data updates -->
<Button Content="Edit Filter"
        uen:Navigation.Request="Filter"
        uen:Navigation.Data="{Binding CurrentFilter, Mode=TwoWay}" />
```

```xml
<!-- WRONG: Navigation.Data without Navigation.Request — data is ignored -->
<Button Content="View Details"
        uen:Navigation.Data="{Binding SelectedProduct}" />
<!-- Missing Navigation.Request means no navigation occurs -->
```

---

## ResultDataViewMap: GetDataAsync Pattern

**Impact**: HIGH — The standard pattern for "navigate, pick/edit, return result" workflows.

### Registration

```csharp
// Register with input data type and result type
views.Register(
    new ResultDataViewMap<FilterPage, FilterViewModel, FilterCriteria, FilterResult>()
    //                     View       ViewModel       InputData         ResultType
);
```

### Calling Side: GetDataAsync

```csharp
// CORRECT: Navigate and await the result
public async Task OpenFilter()
{
    var result = await _navigator.GetDataAsync<FilterResult>(this, data: _currentFilter);
    if (result is not null)
    {
        ApplyFilter(result);
    }
    // result is null if user navigated back without providing a result
}
```

### Target Side: NavigateBackWithResultAsync

```csharp
// CORRECT: Target ViewModel returns data when done
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
        var result = new FilterResult(
            Category: SelectedCategory,
            MinPrice: MinPrice,
            MaxPrice: MaxPrice
        );
        await _navigator.NavigateBackWithResultAsync(this, data: result);
    }

    public async Task Cancel()
    {
        // Navigate back without result — caller receives null
        await _navigator.NavigateBackAsync(this);
    }
}
```

```csharp
// WRONG: Using NavigateBackAsync instead of NavigateBackWithResultAsync
// The caller's GetDataAsync returns null even though a result was available
await _navigator.NavigateBackAsync(this);
// Should be: await _navigator.NavigateBackWithResultAsync(this, data: result);
```

---

## Qualifiers: Controlling Back Stack Behavior

**Impact**: HIGH — Qualifiers control what happens to the navigation history during navigation.

### Qualifier Table

| Qualifier | XAML Prefix | C# Constant | Behavior |
|---|---|---|---|
| Back | `-` | `Qualifiers.NavigateBack` | Navigate back one level |
| Clear back stack | `-/` | `Qualifiers.ClearBackStack` | Remove all entries and navigate |
| Remove current + navigate | `-RouteName` | N/A | Remove current page, then navigate to route |
| Dialog | `!` | `Qualifiers.Dialog` | Open route as a dialog/flyout |

### XAML Usage

```xml
<!-- Back: navigate back one level -->
<Button Content="Back" uen:Navigation.Request="-" />

<!-- Clear back stack: remove all history and navigate to Home -->
<Button Content="Continue" uen:Navigation.Request="-/Home" />

<!-- Open as dialog -->
<Button Content="Filter" uen:Navigation.Request="!Filter" />

<!-- Remove current page from stack, then navigate to Dashboard -->
<Button Content="Skip Intro" uen:Navigation.Request="-Dashboard" />
```

### C# Usage

```csharp
// Back
await navigator.NavigateBackAsync(this);

// Clear back stack
await navigator.NavigateViewModelAsync<HomeViewModel>(this, qualifier: Qualifiers.ClearBackStack);

// Dialog
await navigator.NavigateViewModelAsync<FilterViewModel>(this, qualifier: Qualifiers.Dialog);

// Clear back stack with route name
await navigator.NavigateRouteAsync(this, "Home", qualifier: Qualifiers.ClearBackStack);
```

---

## Common Use Case: Post-Login Navigation

**Impact**: HIGH — After login, you MUST clear the back stack to prevent the user from pressing Back to return to the login page.

```csharp
// CORRECT: After successful login, clear back stack
public async Task OnLoginSuccess()
{
    await _navigator.NavigateViewModelAsync<MainViewModel>(
        this,
        qualifier: Qualifiers.ClearBackStack
    );
}
```

```xml
<!-- CORRECT: XAML equivalent with -/ prefix -->
<Button Content="Sign In" uen:Navigation.Request="-/Main" />
```

```csharp
// WRONG: Regular navigation after login — user can press Back to reach login screen
public async Task OnLoginSuccess()
{
    await _navigator.NavigateViewModelAsync<MainViewModel>(this);
    // User presses Back → returns to LoginPage!
}
```

---

## Combining Data and Qualifiers

```csharp
// CORRECT: Navigate with data AND clear back stack
await navigator.NavigateDataAsync(this, userProfile, qualifier: Qualifiers.ClearBackStack);

// CORRECT: Open detail as dialog with data
await navigator.NavigateDataAsync(this, selectedProduct, qualifier: Qualifiers.Dialog);
```

```xml
<!-- CORRECT: XAML — clear back stack and pass data -->
<Button Content="Go to Dashboard"
        uen:Navigation.Request="-/Dashboard"
        uen:Navigation.Data="{Binding UserProfile}" />
```

---

## Reference

- [Passing Navigation Data (Chefs)](https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/PassingNavigationData.html)
- [Display Item Details Walkthrough](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/DisplayItemDetails.html)
- [Navigation Qualifiers Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Navigation/Qualifiers.html)
- [Advanced Page Navigation](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/AdvancedPageNavigation.html)
- [XAML Navigation (Chefs)](https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/XamlNavigation.html)
