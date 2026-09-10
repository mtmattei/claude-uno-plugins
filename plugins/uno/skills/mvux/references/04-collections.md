# Collections and Selection

How to use `IListFeed<T>` for read-only collections, `IListState<T>` for mutable collections, and implement selection tracking.

---

## IListFeed\<T\> — Read-Only Collections

**Impact**: HIGH — The standard way to display service-loaded lists.

### Creating a ListFeed

```csharp
// From an async method (most common)
public IListFeed<Person> People => ListFeed.Async(PeopleService.GetPeopleAsync);

// From an IAsyncEnumerable
public IListFeed<Person> People => ListFeed.AsyncEnumerable(PeopleService.StreamPeople);
```

Service methods should return `ValueTask<IImmutableList<T>>` (from `System.Collections.Immutable`):

```csharp
public interface IPeopleService
{
    ValueTask<IImmutableList<Person>> GetPeopleAsync(CancellationToken ct);
}
```

### ListFeed Operators

Operators work on **individual items**, not the list itself:

```csharp
// Filter items
public IListFeed<string> LongNames => Names.Where(name => name.Length >= 10);

// Convert between feed types
IFeed<IImmutableCollection<T>> feed = listFeed.AsFeed();
IListFeed<T> listFeed = feed.AsListFeed();
```

### Empty Collections = None

When a `ListFeed` returns an empty collection or `null`, it transitions to `None` state (shows `NoneTemplate` in `FeedView`). This is by design — an empty `ListView` with no items should show "No results" rather than a blank list.

### Binding to ListView

```xml
<!-- Direct binding — no FeedView needed for simple cases -->
<ListView ItemsSource="{Binding People}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="5">
                <TextBlock Text="{Binding FirstName}" />
                <TextBlock Text="{Binding LastName}" />
            </StackPanel>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>

<!-- With FeedView — for loading/error/none states -->
<mvux:FeedView Source="{Binding People}">
    <DataTemplate>
        <ListView ItemsSource="{Binding Data}" />
    </DataTemplate>
</mvux:FeedView>
```

---

## IListState\<T\> — Mutable Collections

**Impact**: HIGH — Required for add/remove/update operations.

### Creating a ListState

```csharp
// Empty
public IListState<string> Items => ListState<string>.Empty(this);

// With initial sync value
public IListState<string> Favorites => ListState.Value(this, () => _favorites);

// From async source
public IListState<Person> People => ListState.Async(this, PeopleService.GetAllAsync);

// From an existing feed
public IListFeed<string> FavoritesFeed => ...
public IListState<string> FavoritesState => ListState.FromFeed(this, FavoritesFeed);
```

### Mutation Operations

```csharp
// Add to end
await MyItems.AddAsync("New Item", ct);

// Insert at beginning
await MyItems.InsertAsync("First Item", ct);

// Remove by predicate
await MyItems.RemoveAllAsync(item => item.Contains("old"), ct);

// Update all items (bulk transform)
await MyItems.Update(
    updater: existing => existing.Select(item => item.Trim()).ToImmutableList(),
    ct: ct);

// Update matching items
await MyItems.UpdateAllAsync(
    match: item => item?.Length > 10,
    updater: item => item.Trim(),
    ct: ct);

// Update by key (requires IKeyEquatable<T>)
await MyItems.UpdateItemAsync(
    oldItem: existingItem,
    updater: item => item with { Value = item.Value.ToUpper() },
    ct: ct);

// Replace by key
await MyItems.UpdateItemAsync(
    oldItem: existingItem,
    newItem: new MyItem(2, "TWO"),
    ct: ct);
```

### ForEach on ListState

Executes once for the entire collection when it changes:

```csharp
await MyStrings.ForEach(async (list, ct) => await ProcessAll(list, ct));
```

---

## Selection

**Impact**: HIGH — Built-in selection tracking for ListView/GridView.

### Single-Item Selection

```csharp
public partial record PeopleModel(IPeopleService PeopleService)
{
    public IListFeed<Person> People =>
        ListFeed
            .Async(PeopleService.GetPeopleAsync)
            .Selection(SelectedPerson);  // Link selection to state

    public IState<Person> SelectedPerson => State<Person>.Empty(this);
}
```

```xml
<StackPanel DataContext="{Binding SelectedPerson}" Orientation="Horizontal">
    <TextBlock Text="Selected: " />
    <TextBlock Text="{Binding FirstName}" />
</StackPanel>

<ListView ItemsSource="{Binding People}" SelectionMode="Single">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding FirstName}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

Selection sync happens automatically — no need to bind `ListView.SelectedItem`.

### Multi-Item Selection

```csharp
public partial record PeopleModel(IPeopleService PeopleService)
{
    public IListFeed<Person> People =>
        ListFeed
            .Async(PeopleService.GetPeopleAsync)
            .Selection(SelectedPeople);

    public IState<IImmutableList<Person>> SelectedPeople =>
        State<IImmutableList<Person>>.Empty(this);
}
```

Set `SelectionMode="Multiple"` on the `ListView`.

### Selection into a Related Property

Project a selected item's key into an aggregate object:

```csharp
public record Profile(string? FirstName, string? CountryId);
public record Country(string Id, string Name);

public IState<Profile> Profile => State.Async(this, _svc.GetProfile);
public IListState<Country> Countries =>
    ListState.Async(this, _svc.GetCountries)
        .Selection(Profile, p => p.CountryId);
```

```xml
<mvux:FeedView Source="{Binding Countries}">
    <DataTemplate>
        <ComboBox ItemsSource="{Binding Data}" />
    </DataTemplate>
</mvux:FeedView>
```

### Listening to Selection Changes

**With Select operator:**

```csharp
public IFeed<string> Greeting =>
    SelectedPerson.Select(p => p is null ? "" : $"Hello {p.FirstName}!");
```

**With ForEach:**

```csharp
public PeopleModel(IPeopleService peopleService)
{
    _peopleService = peopleService;
    SelectedPerson.ForEach(OnSelectionChanged);
}

private async ValueTask OnSelectionChanged(Person? person, CancellationToken ct)
{
    if (person is null) return;
    await Greeting.SetAsync($"Hello {person.FirstName}!", ct);
}
```

**Via command parameter:**

```csharp
// Parameter name matches SelectedPerson state — auto-resolved
public ValueTask CheckSelection(Person selectedPerson)
{
    // selectedPerson is the current selection
}
```

### Manual Selection (Programmatic)

Use `IListState<T>` (not `IListFeed<T>`) for programmatic selection:

```csharp
IListState<string> Names => ...

// Select a single item
await Names.TrySelectAsync("charlie", ct);

// Select multiple items
await Names.TrySelectAsync(ImmutableList.Create("charlie", "joe"), ct);

// Clear selection
await Names.ClearSelection(ct);
```

---

## ListFeed vs ListState Decision

| Need | Use |
|---|---|
| Display read-only list from service | `IListFeed<T>` |
| Add/remove/edit items | `IListState<T>` |
| Track selection (read-only source) | `IListFeed<T>.Selection(state)` |
| Programmatic selection | `IListState<T>` |
| CRUD with messaging sync | `IListState<T>` + `Observe()` |

---

## Reference

- [ListFeed Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Reactive/listfeed.html)
- [ListState Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/ListStates.html)
- [Selection Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Advanced/Selection.html)
- [Usage in Apps](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Reactive/in-apps.html)
