# Advanced Patterns

Pagination (incremental loading, offset, cursor-based) and other advanced MVUX patterns.

---

## Incremental Loading (Automatic Pagination)

**Impact**: HIGH — The simplest way to load large datasets incrementally.

Uses `ISupportIncrementalLoading` built into `ListView`/`GridView`. When the user scrolls to the bottom, the next page loads automatically.

### Service

```csharp
public interface IPeopleService
{
    ValueTask<IImmutableList<Person>> GetPeopleAsync(
        uint pageSize, uint firstItemIndex, CancellationToken ct);
}
```

### Model

```csharp
public partial record PeopleModel(IPeopleService PeopleService)
{
    const uint DefaultPageSize = 20;

    public IListFeed<Person> People =>
        ListFeed.AsyncPaginated(async (PageRequest pageRequest, CancellationToken ct) =>
            await PeopleService.GetPeopleAsync(
                pageSize: pageRequest.DesiredSize ?? DefaultPageSize,
                firstItemIndex: pageRequest.CurrentCount,
                ct));
}
```

### View

```xml
<ListView ItemsSource="{Binding People}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="2">
                <TextBlock Text="{Binding FirstName}" />
                <TextBlock Text="{Binding LastName}" />
            </StackPanel>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

No special XAML — `ListView` handles incremental loading automatically via the generated ViewModel.

### PageRequest Properties

| Property | Type | Description |
|---|---|---|
| `DesiredSize` | `uint?` | Items the view wants to display. **null on first call** — use a default. |
| `CurrentCount` | `uint` | Total items already loaded (use as offset). |
| `Index` | `uint` | Zero-based page index. |

**Warning**: `DesiredSize` may change between pages (e.g., if user resizes the app). Do NOT use `Index * DesiredSize` for skip/take. Instead use `CurrentCount` as the offset:

```csharp
// CORRECT
source.Skip(pageRequest.CurrentCount).Take(pageRequest.DesiredSize ?? 20)

// WRONG — DesiredSize can change between pages
source.Skip(pageRequest.Index * pageRequest.DesiredSize).Take(pageRequest.DesiredSize)
```

### Loading Indicator in Footer

```xml
<mvux:FeedView Source="{Binding People}">
    <DataTemplate>
        <ListView ItemsSource="{Binding Data}">
            <ListView.Footer>
                <ProgressBar
                    Visibility="{Binding Data.ExtendedProperties.IsLoadingMoreItems}"
                    IsIndeterminate="{Binding Data.ExtendedProperties.IsLoadingMoreItems}"
                    HorizontalAlignment="Stretch" />
            </ListView.Footer>
        </ListView>
    </DataTemplate>
</mvux:FeedView>
```

---

## Offset Pagination (Manual Page Control)

**Impact**: MEDIUM — When you need explicit page navigation (prev/next buttons, page number).

### Model

```csharp
public partial record PeopleModel(IPeopleService PeopleService)
{
    const uint DefaultPageSize = 20;

    public IFeed<uint> PageCount =>
        Feed.Async(async ct => await PeopleService.GetPageCount(DefaultPageSize, ct));

    public IState<uint> CurrentPage => State.Value(this, () => 1u);

    public IListFeed<Person> People =>
        CurrentPage
            .SelectAsync(async (currentPage, ct) =>
                await PeopleService.GetPeopleAsync(
                    pageSize: DefaultPageSize,
                    firstItemIndex: (currentPage - 1) * DefaultPageSize, ct))
            .AsListFeed();
}
```

The `People` feed depends on `CurrentPage` state — it auto-refreshes when the page number changes.

### View

```xml
<ListView ItemsSource="{Binding People}">
    <ListView.Footer>
        <NumberBox
            HorizontalAlignment="Center"
            Header="Current page:"
            Minimum="1"
            Maximum="{Binding PageCount}"
            SpinButtonPlacementMode="Inline"
            Value="{Binding CurrentPage, Mode=TwoWay}" />
    </ListView.Footer>
</ListView>
```

---

## Cursor-Based (Keyset) Pagination

**Impact**: MEDIUM — More efficient than offset for large datasets; avoids skip/duplicate issues.

### Service

```csharp
public interface IPeopleService
{
    ValueTask<(IImmutableList<Person> CurrentPage, int? NextCursor)> GetPeopleAsync(
        int? cursor, uint pageSize, CancellationToken ct);
}
```

The service returns the page items AND a cursor pointing to the start of the next page. Return `null` cursor when there are no more pages.

### Model

```csharp
public IListFeed<Person> People =>
    ListFeed<Person>.AsyncPaginatedByCursor(
        firstPage: default(int?),  // Starting cursor (null = beginning)
        getPage: async (cursor, desiredPageSize, ct) =>
        {
            var result = await PeopleService.GetPeopleAsync(
                cursor, desiredPageSize ?? 20, ct);
            return new PageResult<int?, Person>(
                result.CurrentPage, result.NextCursor);
        });
```

### PageResult\<TCursor, TItem\>

```csharp
// Constructor: items for this page + cursor for next page
new PageResult<int?, Person>(items, nextCursor)
```

If `nextCursor` is `null` (or the default for the cursor type), pagination stops.

### View

Same as incremental loading — `ListView` handles it automatically:

```xml
<ListView ItemsSource="{Binding People}" />
```

---

## ItemsRepeater with Pagination

`ItemsRepeater` doesn't support `ISupportIncrementalLoading` natively. Use the Toolkit's `ItemsRepeaterExtensions`:

```xml
<muxc:ItemsRepeater
    ItemsSource="{Binding People}"
    utu:ItemsRepeaterExtensions.SupportsIncrementalLoading="True">
    <muxc:ItemsRepeater.Layout>
        <muxc:StackLayout />
    </muxc:ItemsRepeater.Layout>
</muxc:ItemsRepeater>
```

Requires: `Uno.Toolkit.UI` NuGet package.

---

## Combining Patterns

### Feed Dependent on Multiple States

```csharp
public IState<string> SearchQuery => State<string>.Empty(this);
public IState<string> Category => State.Value(this, () => "All");

public IListFeed<Product> SearchResults =>
    SearchQuery.SelectAsync(async (query, ct) =>
    {
        var cat = await Category;
        return await _productService.Search(query, cat, ct);
    }).AsListFeed();
```

### Refresh with Signal

For manual refresh triggers beyond FeedView's built-in Refresh:

```csharp
public IState<string> City => State.Async(this, async ct =>
    await _locationService.GetCurrentCity(ct),
    refreshSignal: _refreshSignal);

private readonly Signal _refreshSignal = new();

public async ValueTask ForceRefresh() => _refreshSignal.Raise();
```

---

## Common Mistakes

- **Using `DesiredSize` for skip/take offset**: It changes between pages — use `CurrentCount` instead
- **Not handling null `DesiredSize` on first call**: Always provide a default page size
- **Cursor pagination with wrong `firstPage`**: Must match the cursor type expected by your service
- **Forgetting `.AsListFeed()`**: When building a `IListFeed` from a state's `SelectAsync`, the result is `IFeed<IImmutableList<T>>` — call `.AsListFeed()` to convert

---

## Reference

- [Pagination Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Advanced/Pagination.html)
- [ItemsRepeater Extensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/walkthroughs/itemsrepeater-extensions.howto.html)
- [Usage in Apps](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Reactive/in-apps.html)
