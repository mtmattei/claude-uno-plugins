---
name: silverlight-to-uno-migration
description: "Migrate Silverlight applications to Uno Platform. Covers URI-based Page/Frame navigation to Navigation Extensions, WCF RIA Services to Kiota/Refit HTTP clients, IsolatedStorage to ApplicationData, ChildWindow to dialog navigation, and Silverlight Toolkit control mapping. Use when: (1) Planning or executing a Silverlight-to-Uno migration, (2) Replacing NavigationService/NavigationContext.QueryString patterns, (3) Finding a .NET 9 replacement for DomainService/DomainContext, (4) Replacing IsolatedStorage, (5) Mapping Silverlight Toolkit controls (Accordion, AutoCompleteBox, BusyIndicator) to WinUI/Uno equivalents. Do NOT use for: WPF migration (see wpf-to-uno-migration), assessing migration readiness (see wpf-migration-assessment), build or runtime errors during migration (see uno-build-troubleshoot), or new project setup (see uno-platform-agent)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation)"
metadata:
  version: "2.0.0"
---

# Silverlight to Uno Platform Migration

Silverlight has no supported .NET 9 runtime and no direct WinUI equivalent for its
three defining subsystems. Migration is an architecture change, not an API swap.

## Scope

This skill owns the Silverlight-specific subsystems. Everything a Silverlight app
shares with WPF — `System.Windows.*` namespaces, `x:Static`, `MultiBinding`,
`Style.Triggers`, general XAML and control mapping — lives in
[`wpf-to-uno-migration`](../wpf-to-uno-migration/SKILL.md). Load both for a
Silverlight app; they compose.

## The Three Breaking Subsystems

| Silverlight | Status | Uno Platform replacement |
|---|---|---|
| `NavigationService` + URI fragments | No equivalent | Navigation Extensions, typed routes |
| `NavigationContext.QueryString` | No equivalent | `Navigation.Data` |
| WCF RIA Services (`DomainService`/`DomainContext`) | Dead, no .NET 9 support | Kiota (OpenAPI) or Refit |
| `IsolatedStorage` | No equivalent | `ApplicationData.Current.LocalFolder` |
| `ChildWindow` | No equivalent | Dialog navigation via `Navigation.Request` |
| Silverlight Toolkit controls | Partial | WinUI built-ins + Uno Toolkit |

## Navigation

Silverlight navigates by URI (`/Views/OrderDetails.xaml?orderId=123`) and reads
parameters from a query-string bag. Uno Platform registers typed routes and injects
data into the ViewModel.

```csharp
// Silverlight
NavigationService.Navigate(new Uri("/Views/OrderDetails.xaml?orderId=123", UriKind.Relative));
var orderId = NavigationContext.QueryString["orderId"];   // in OnNavigatedTo

// Uno Platform
views.Register(new ViewMap<OrderDetailsPage, OrderDetailsViewModel>());
routes.Register(new RouteMap("OrderDetails", View: views.FindByViewModel<OrderDetailsViewModel>()));
public partial record OrderDetailsViewModel(int OrderId);
```

Do not try to reproduce the query-string bag. Pass data with `Navigation.Data` and
let DI construct the ViewModel.

## Data Access

WCF RIA Services has no migration target — the data layer is rewritten as HTTP.
Kiota for OpenAPI-generated clients, Refit for hand-declared contracts. Both
register through `UseHttp()`.

```csharp
host.UseHttp((context, services) => services.AddRefitClient<IOrderApi>(context));
```

Pair with MVUX feeds rather than the `Load()`/callback pattern:

```csharp
public IListFeed<Order> Orders => ListFeed.Async(async ct => await ApiClient.Orders.GetAsync(ct));
```

## Storage

`IsolatedStorage` becomes `ApplicationData.Current.LocalFolder`. Do not substitute
`System.IO.File` — it works on Desktop and fails on mobile and WebAssembly due to
sandboxing. On Wasm the folder maps to IndexedDB, with async semantics and size limits.

## Common Mistakes

- Searching for a .NET 9 RIA Services replacement; the architecture must change.
- Reproducing `NavigationContext.QueryString` through static state or singletons.
- Wrapping HTTP calls synchronously to mimic `Load()` instead of async/await.
- Migrating `ChildWindow` as a custom popup instead of dialog navigation.
- Assuming every Silverlight Toolkit control has a 1:1 equivalent.
- Using `System.IO.File` for local storage.

## Related Skills

| Skill | Use instead when... |
|---|---|
| `wpf-to-uno-migration` | Migrating WPF, or handling the WPF-shared XAML/API surface of a Silverlight app |
| `wpf-migration-assessment` | Scoring readiness, effort, and risk before committing |
| `uno-build-troubleshoot` | The migration builds badly or crashes at runtime |
| `uno-navigation` | Navigation Extensions in depth, beyond the Silverlight mapping |
| `uno-extensions-services` | Kiota/Refit, DI, and configuration in depth |

## Detailed References

- [references/01-silverlight-patterns.md](references/01-silverlight-patterns.md) - Read for full worked examples of the three subsystems, with rule/why/example/mistakes for each and links to the Uno Platform docs
