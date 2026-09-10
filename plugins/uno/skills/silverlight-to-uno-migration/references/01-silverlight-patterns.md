# Silverlight to Uno Platform Migration Reference

Detailed patterns for the three Silverlight subsystems with no WinUI equivalent:
navigation, RIA Services, and IsolatedStorage/Toolkit controls.

### Migrate Silverlight Page/Frame Navigation

**Rule**: Replace Silverlight `Page`/`Frame` navigation with Uno Platform Navigation Extensions; map Silverlight's URI-based navigation to typed route registrations.

**Why**: Silverlight navigation relies on URI fragments (`#/page/id`) and a built-in `NavigationService`. This model does not exist in WinUI or Uno Platform. Uno Navigation Extensions provide an equivalent declarative navigation system with type safety, dependency injection integration, and cross-platform back-button support that Silverlight's model lacked.

**Example (Silverlight - URI-based navigation)**:
```csharp
// Silverlight pattern
NavigationService.Navigate(new Uri("/Views/OrderDetails.xaml?orderId=123", UriKind.Relative));

// In the target page:
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    var orderId = NavigationContext.QueryString["orderId"];
}
```

**Example (Uno Platform - typed navigation)**:
```csharp
// Route registration
views.Register(new ViewMap<OrderDetailsPage, OrderDetailsViewModel>());
routes.Register(
    new RouteMap("OrderDetails", View: views.FindByViewModel<OrderDetailsViewModel>())
);

// ViewModel receives data via constructor injection
public partial record OrderDetailsViewModel(int OrderId);
```

```xml
<!-- XAML navigation with data -->
<Button Content="View Order"
        uen:Navigation.Request="OrderDetails"
        uen:Navigation.Data="{x:Bind ViewModel.SelectedOrderId}" />
```

**Common Mistakes**:
- Trying to replicate Silverlight's `NavigationContext.QueryString` pattern (does not exist in Uno Platform)
- Passing data through static variables or singletons instead of using Navigation.Data
- Not registering ViewModels with the DI container, causing resolution failures during navigation
- Keeping Silverlight-style `OnNavigatedTo` overrides for data loading instead of using MVUX Feeds or async ViewModel initialization

**Uno Platform Notes**: Silverlight's child window (popup) pattern maps to Uno Platform's dialog navigation. Use `Navigation.Request` with a route that is registered as a dialog to get modal behavior equivalent to Silverlight's `ChildWindow`.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Navigation/NavigationOverview.html

---

### Replace Silverlight RIA Services with Modern HTTP Clients

**Rule**: Replace WCF RIA Services (DomainService, DomainContext, EntityQuery) with Kiota-generated or Refit-based HTTP clients backed by REST or GraphQL APIs.

**Why**: WCF RIA Services is a Silverlight-era technology with no .NET 9 support and no WinUI equivalent. Uno Platform recommends Kiota (for OpenAPI/REST) or Refit (for typed REST) as modern replacements. These integrate with Uno.Extensions dependency injection and support all target platforms including WebAssembly.

**Example (Silverlight RIA - legacy pattern)**:
```csharp
// Silverlight RIA Services - NOT SUPPORTED
var context = new OrderDomainContext();
var query = context.GetOrdersQuery();
context.Load(query, LoadBehavior.MergeIntoCurrent, OnOrdersLoaded, null);

private void OnOrdersLoaded(LoadOperation<Order> operation)
{
    if (!operation.HasError)
    {
        OrdersList.ItemsSource = operation.Entities;
    }
}
```

**Example (Uno Platform - Kiota HTTP client)**:
```csharp
// Service registration in App.xaml.cs
host.ConfigureServices((context, services) =>
{
    services.AddHttpClient<IOrderApiClient, OrderApiClient>(client =>
    {
        client.BaseAddress = new Uri("https://api.example.com");
    });
});

// ViewModel with MVUX Feed
public partial record OrderListViewModel(IOrderApiClient ApiClient)
{
    public IListFeed<Order> Orders => ListFeed.Async(
        async ct => await ApiClient.Orders.GetAsync(cancellationToken: ct)
    );
}
```

**Example (Uno Platform - Refit)**:
```csharp
// Define the API interface
public interface IOrderApi
{
    [Get("/api/orders")]
    Task<List<Order>> GetOrdersAsync(CancellationToken ct = default);

    [Get("/api/orders/{id}")]
    Task<Order> GetOrderAsync(int id, CancellationToken ct = default);
}

// Registration
host.UseHttp((context, services) =>
{
    services.AddRefitClient<IOrderApi>(context);
});
```

**Common Mistakes**:
- Trying to find a .NET 9 WCF RIA Services replacement (none exists; the architecture must change)
- Creating synchronous HTTP wrappers to mimic RIA Services' `Load()` pattern instead of using async/await
- Not handling cancellation tokens in HTTP calls, causing wasm timeouts and mobile battery drain
- Hardcoding API base URLs instead of using configuration-based endpoints

**Uno Platform Notes**: Kiota is the preferred HTTP client for new Uno Platform projects because it generates strongly-typed clients from OpenAPI specifications. Refit is recommended when you want to define the API contract manually as a C# interface. Both integrate with Uno.Extensions `UseHttp()` for DI registration.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Http/HttpOverview.html

---

### Replace Silverlight IsolatedStorage and Toolkit Controls

**Rule**: Replace `IsolatedStorage` with `ApplicationData.Current.LocalFolder` and map Silverlight Toolkit controls to their Uno Toolkit equivalents.

**Why**: `IsolatedStorage` was Silverlight's sandboxed file storage API. It does not exist in WinUI or Uno Platform. `ApplicationData.Current.LocalFolder` provides equivalent cross-platform local storage with proper sandboxing on all targets. Similarly, Silverlight Toolkit controls (Accordion, AutoCompleteBox, Rating, etc.) have no direct WinUI equivalent but many have Uno Toolkit counterparts.

**Example (IsolatedStorage replacement)**:
```csharp
// Silverlight - NOT SUPPORTED
using System.IO.IsolatedStorage;

var store = IsolatedStorageFile.GetUserStoreForApplication();
using var stream = store.CreateFile("settings.json");
// write data

// Uno Platform - cross-platform local storage
var localFolder = Windows.Storage.ApplicationData.Current.LocalFolder;
var file = await localFolder.CreateFileAsync("settings.json",
    CreationCollisionOption.ReplaceExisting);
await FileIO.WriteTextAsync(file, jsonContent);
```

**Example (Toolkit control mapping)**:

| Silverlight Toolkit | Uno Platform Equivalent |
|---|---|
| `Accordion` | Custom expander list using `Expander` control |
| `AutoCompleteBox` | `AutoSuggestBox` (built-in WinUI control) |
| `BusyIndicator` | `LoadingView` (Uno Toolkit) |
| `DatePicker` / `TimePicker` | `DatePicker` / `TimePicker` (built-in WinUI) |
| `NumericUpDown` | `NumberBox` (built-in WinUI) |
| `Rating` | `RatingControl` (built-in WinUI) |
| `TabControl` | `TabBar` (Uno Toolkit) or `TabView` (WinUI) |
| `TreeView` | `TreeView` (built-in WinUI) |
| `WrapPanel` | `ItemsWrapGrid` or Uno Toolkit `AutoLayout` with wrapping |
| `ChildWindow` | Dialog navigation via Navigation Extensions |

**Common Mistakes**:
- Using `System.IO.File` directly instead of `ApplicationData.Current.LocalFolder` (works on Desktop but fails on mobile/wasm due to sandboxing)
- Assuming all Silverlight Toolkit controls have 1:1 equivalents (some require custom implementations)
- Not testing storage APIs on WebAssembly (local storage maps to IndexedDB, with different size limits)
- Migrating Silverlight `ChildWindow` as a custom popup instead of using Uno Platform's dialog navigation

**Uno Platform Notes**: For WebAssembly, `ApplicationData.Current.LocalFolder` maps to the browser's IndexedDB storage. File operations are asynchronous and have platform-specific size limits. For large data sets, consider using Uno.Extensions Storage or a client-side database like LiteDB.

**Reference**: https://platform.uno/docs/articles/features/windows-storage.html
