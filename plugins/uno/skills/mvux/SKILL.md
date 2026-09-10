---
name: mvux
description: "MVUX (Model-View-Update-eXtended) architecture for Uno Platform: immutable records, reactive feeds/states, FeedView, commands, selection, messaging, and pagination. Use when: (1) Creating MVUX models with IFeed/IState/IListFeed/IListState, (2) Displaying async data with FeedView, (3) Implementing two-way binding with states, (4) Working with commands, selection, or messaging, (5) Debugging blank FeedView or missing command generation. Do NOT use for: MVVM patterns (use CommunityToolkit.Mvvm directly), general XAML layout (see winui-xaml), navigation (see uno-navigation)."
license: "MIT"
metadata:
  version: "1.0.0"
  author: "Uno Platform"
  category: "uno-platform-mvux"
  tags: [mvux, reactive, feeds, states, uno-platform, immutable]
---

# MVUX — Model-View-Update-eXtended

Reactive architecture for Uno Platform apps. Models are immutable `partial record` types that expose `IFeed<T>` (read-only) and `IState<T>` (read-write) properties. A ViewModel is code-generated from each Model to bridge immutable data with XAML data binding.

## Quick Reference

1. Models MUST be `partial record` with a `Model` suffix (e.g., `MainModel`) — the generator creates `MainViewModel`
2. Data entities SHOULD be `record` types (immutable, value equality)
3. Use `IFeed<T>` for read-only async data, `IState<T>` for mutable/two-way bound data
4. Use `IListFeed<T>` for read-only collections, `IListState<T>` for mutable collections
5. Set `DataContext` to the **generated ViewModel**, not the Model: `new MainViewModel(...)`
6. Inside `FeedView` templates, access data via `{Binding Data.PropertyName}`
7. Public async methods on the Model auto-generate `IAsyncCommand` properties on the ViewModel
8. States require `this` as owner for lifecycle management: `State<string>.Empty(this)`
9. Records with a property named `Id` or `Key` auto-generate `IKeyEquatable<T>` — needed for messaging and selection
10. Requires `<UnoFeatures>MVUX</UnoFeatures>` in the project file

## Key Concepts

### MVUX vs MVVM

| Aspect | MVVM | MVUX |
|---|---|---|
| Data flow | Bidirectional | Unidirectional (immutable → generated ViewModel bridges binding) |
| Model type | `class` with `INotifyPropertyChanged` | `partial record` (immutable) |
| State management | Manual property change notifications | Automatic via `IFeed<T>` / `IState<T>` |
| Async states | Manual (loading flags, try/catch) | Built-in (loading/error/none/value) |
| Commands | Manual `ICommand` implementations | Auto-generated from public methods |
| Collections | `ObservableCollection<T>` | `IListFeed<T>` / `IListState<T>` |

### Type Decision Tree

```
Need to display data from a service?
├── Single item, read-only? → IFeed<T>
├── Single item, user can edit? → IState<T>
├── Collection, read-only? → IListFeed<T>
└── Collection, user can add/remove/edit? → IListState<T>
```

### Project Setup

MVUX requires the `MVUX` UnoFeature in your project file:

```xml
<UnoFeatures>MVUX</UnoFeatures>
```

This pulls in the `Uno.Extensions.Reactive` packages and enables the source generator.

### Model Structure

```csharp
// Model: partial record, Model suffix, services via constructor
public partial record MainModel(IWeatherService WeatherService)
{
    // Read-only feed — calls service, handles loading/error automatically
    public IFeed<WeatherInfo> CurrentWeather =>
        Feed.Async(WeatherService.GetCurrentWeather);

    // Mutable state — supports two-way binding
    public IState<string> City => State<string>.Empty(this);

    // Feed dependent on state — auto-refreshes when City changes
    public IFeed<WeatherInfo> CityWeather =>
        City.SelectAsync(WeatherService.GetWeatherForCity);

    // Auto-generates a command on the ViewModel
    public async ValueTask Refresh(CancellationToken ct)
    {
        await City.UpdateAsync(_ => "Montréal", ct);
    }
}
```

```xml
<!-- View: bind to generated ViewModel -->
<Page xmlns:mvux="using:Uno.Extensions.Reactive.UI">
    <mvux:FeedView Source="{Binding CurrentWeather}">
        <DataTemplate>
            <TextBlock Text="{Binding Data.Temperature}" />
        </DataTemplate>
    </mvux:FeedView>
</Page>
```

```csharp
// Code-behind: set DataContext to generated ViewModel
public MainPage()
{
    InitializeComponent();
    DataContext = new MainViewModel(new WeatherService());
}
```

## Critical Patterns

### FeedView with All Templates

```xml
<!-- CORRECT: FeedView handles all async states -->
<mvux:FeedView Source="{Binding CurrentWeather}">
    <mvux:FeedView.ValueTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Data.Temperature}" />
        </DataTemplate>
    </mvux:FeedView.ValueTemplate>
    <mvux:FeedView.ProgressTemplate>
        <DataTemplate>
            <ProgressRing />
        </DataTemplate>
    </mvux:FeedView.ProgressTemplate>
    <mvux:FeedView.ErrorTemplate>
        <DataTemplate>
            <TextBlock Text="An error occurred" />
        </DataTemplate>
    </mvux:FeedView.ErrorTemplate>
    <mvux:FeedView.NoneTemplate>
        <DataTemplate>
            <TextBlock Text="No data available" />
        </DataTemplate>
    </mvux:FeedView.NoneTemplate>
</mvux:FeedView>
```

```xml
<!-- WRONG: binding directly without FeedView — no loading/error handling -->
<TextBlock Text="{Binding CurrentWeather.Temperature}" />
```

### Two-Way Binding with IState

```csharp
// CORRECT: IState for user input
public IState<string> City => State<string>.Empty(this);
```

```xml
<!-- CORRECT: two-way binding works automatically with generated ViewModel -->
<TextBox Text="{Binding City, Mode=TwoWay}" />
```

```csharp
// WRONG: IFeed cannot accept user input
public IFeed<string> City => Feed.Async(async ct => "Montréal");
```

### State Updates Must Be Awaited

```csharp
// CORRECT: await the update
public async ValueTask SetCity(CancellationToken ct)
{
    await City.UpdateAsync(_ => "Toronto", ct);
}

// WRONG: fire-and-forget loses error handling
public void SetCity()
{
    _ = City.UpdateAsync(_ => "Toronto");
}
```

### ListView with IListFeed (No FeedView Needed)

```xml
<!-- CORRECT: ListView can bind directly to a list feed -->
<ListView ItemsSource="{Binding People}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding FirstName}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

### Command with Feed Parameter

```csharp
// The 'counterValue' parameter auto-resolves from the CounterValue feed
// because name and type match (case-insensitive)
public IFeed<int> CounterValue => ...

public async ValueTask ResetCounter(int counterValue, CancellationToken ct)
{
    // counterValue contains the current feed value when the command executes
}
```

```xml
<Button Command="{Binding ResetCounter}" Content="Reset" />
```

## Common Mistakes

**Model declaration:**
- Forgetting `partial` on the record — the source generator cannot produce the ViewModel
- Missing `Model` suffix — the generator won't match the class (configurable via `[ReactiveBindable]` or `[ImplicitBindables]`)
- Setting `DataContext` to the Model instead of the generated ViewModel
- Using `class` instead of `record` — loses immutability guarantees

**Feeds and states:**
- Using `IFeed<T>` when `IState<T>` is needed — feeds are read-only, two-way binding silently does nothing
- Using `IState<T>` when `IFeed<T>` suffices — states hold cached values and are heavier
- Forgetting `this` in state factory methods (`State<T>.Empty(this)`) — required for lifecycle management
- Not awaiting `UpdateAsync` / `SetAsync` — updates may be lost or errors swallowed
- Using `Feed<List<T>>` instead of `IListFeed<T>` — operators won't work on individual items

**FeedView:**
- Binding to `{Binding PropertyName}` inside FeedView templates instead of `{Binding Data.PropertyName}`
- Not providing `ProgressTemplate` — default progress ring may not match your design
- Empty collections show `NoneTemplate`, not `ValueTemplate` — this is by design for `IListFeed<T>`

**Commands:**
- Expecting a command from a method that returns a value (non-void, non-Task/ValueTask) — not generated
- Feed parameter name/type mismatch — the parameter won't be resolved from the feed and will be treated as `CommandParameter`

**Records and key equality:**
- Using `set` instead of `init` on record properties — breaks immutability
- Missing `partial` on data records that need `IKeyEquatable<T>` auto-generation
- No `Id` or `Key` property on records used with messaging — entities can't be matched for updates

## Related Skills

| Skill | Use instead when... |
|---|---|
| `winui-xaml` | Working with XAML layout, binding syntax, or performance (not MVUX-specific) |
| `uno-platform-agent` | Creating a new Uno Platform project from scratch |
| `uno-navigation` | Setting up page navigation and routing |
| `uno-toolkit` | Using Toolkit controls (AutoLayout, SafeArea, TabBar) |
| `uno-material` | Configuring Material Design theme and styling |
| `dotnet-csharp` | General C# best practices not specific to MVUX |

## Detailed References

Read the reference file matching your current task:

- [references/01-records-code-generation.md](references/01-records-code-generation.md) — Read when designing data models, understanding immutability requirements, or configuring key equality
- [references/02-feeds.md](references/02-feeds.md) — Read when creating IFeed<T>, using FeedView, or applying feed operators (Select, Where, SelectAsync)
- [references/03-states.md](references/03-states.md) — Read when implementing two-way binding, updating state programmatically, or subscribing to state changes
- [references/04-collections.md](references/04-collections.md) — Read when working with IListFeed<T>, IListState<T>, or implementing single/multi-item selection
- [references/05-commands-actions.md](references/05-commands-actions.md) — Read when configuring command generation, using feed parameters, messaging, or CRUD sync
- [references/06-advanced-patterns.md](references/06-advanced-patterns.md) — Read when implementing pagination, infinite scroll, cursor-based APIs, or offset pagination

## Docs Fallback

If the encoded knowledge above may be outdated, use MCP to fetch the latest:

```
uno_platform_docs_search("MVUX overview Model View Update")
uno_platform_docs_search("MVUX Feed IFeed async")
uno_platform_docs_search("MVUX State IState mutable binding")
uno_platform_docs_search("MVUX FeedView control display")
uno_platform_docs_search("MVUX commands generation")
uno_platform_docs_search("MVUX ListFeed collection")
uno_platform_docs_search("MVUX ListState mutable collection")
uno_platform_docs_search("MVUX selection")
uno_platform_docs_search("MVUX pagination PaginatedAsync")
uno_platform_docs_search("MVUX messaging EntityMessage")
```
