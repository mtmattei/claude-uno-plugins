# States

How to create `IState<T>` for mutable reactive data, implement two-way binding, and update state programmatically.

---

## Creating States

**Impact**: HIGH — States are required for any user-editable data.

All state factory methods require `this` (the Model instance) as the owner for lifecycle management. States share the same lifetime as their owner — when the View is disposed, states are garbage collected.

### State\<T\>.Empty — No Initial Value

```csharp
public IState<string> City => State<string>.Empty(this);
```

### State.Value — Synchronous Initial Value

```csharp
public IState<string> City => State.Value(this, () => "Montréal");
public IState<double> SliderValue => State.Value(this, () => Random.Shared.NextDouble() * 100);
```

### State.Async — Async Initial Value

```csharp
public IState<string> City =>
    State.Async(this, async ct => await _locationService.GetCurrentCity(ct));
```

### State.AsyncEnumerable — From a Stream

```csharp
public IState<string> City =>
    State.AsyncEnumerable(this, () => GetCurrentCity());

private async IAsyncEnumerable<string> GetCurrentCity(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    while (!ct.IsCancellationRequested)
    {
        yield return await _locationService.GetCurrentCity(ct);
        await Task.Delay(TimeSpan.FromMinutes(15), ct);
    }
}
```

### State.FromFeed — Wrap a Feed as Mutable

```csharp
public IFeed<int> MyFeed => ...
public IState<int> MyState => State.FromFeed(this, MyFeed);
```

---

## Two-Way Binding

**Impact**: HIGH — The main reason to use IState over IFeed.

The generated ViewModel automatically exposes states as two-way bindable properties:

```csharp
public partial record SliderModel
{
    public IState<double> SliderValue => State.Value(this, () => 50.0);
}
```

```xml
<Page>
    <Page.DataContext>
        <local:SliderViewModel />
    </Page.DataContext>

    <StackPanel>
        <TextBlock Text="{Binding SliderValue}" />
        <Slider Value="{Binding SliderValue, Mode=TwoWay}" />
    </StackPanel>
</Page>
```

Moving the Slider instantly updates the TextBlock — no manual `PropertyChanged` needed.

---

## Updating State Programmatically

**Impact**: HIGH — ACID-compliant updates prevent race conditions.

### UpdateAsync — Transform Current Value

The `updater` delegate receives the current value and returns the new value. It may be called more than once for concurrent updates — it must be a **pure function**.

```csharp
public IState<string> City => State<string>.Empty(this);

public async ValueTask SetCurrent(CancellationToken ct)
{
    var city = await _locationService.GetCurrentCity(ct);
    await City.UpdateAsync(currentValue => city, ct);
}
```

```csharp
// Increment pattern
public async ValueTask IncrementSlider(CancellationToken ct = default)
{
    await SliderValue.UpdateAsync(
        updater: current => current <= 99 ? current + 1 : 1,
        ct);
}
```

### SetAsync — Replace Value Directly

When you don't need the current value:

```csharp
public async ValueTask SetSliderMiddle(CancellationToken ct = default)
{
    await SliderValue.SetAsync(50, ct);
}
```

### UpdateDataAsync — Update Value and Metadata

For working with the full `Option<T>`:

```csharp
await City.UpdateDataAsync(currentData =>
{
    return Option.Some("New York");
}, ct);
```

---

## Subscribing to Changes

**Impact**: MEDIUM — React to state changes without polling.

### ForEach — Execute Callback on Change

```csharp
public partial record Model
{
    public IState<string> MyState => State.Value(this, () => "Initial")
        .ForEach(PerformAction);  // Fluent API

    public async ValueTask PerformAction(string? item, CancellationToken ct)
    {
        // Called each time MyState changes
    }
}
```

Or set up in constructor:

```csharp
public partial record PeopleModel
{
    public PeopleModel(IPeopleService peopleService)
    {
        _peopleService = peopleService;
        SelectedPerson.ForEach(OnSelectionChanged);
    }

    public IState<Person> SelectedPerson => State<Person>.Empty(this);

    private async ValueTask OnSelectionChanged(Person? person, CancellationToken ct)
    {
        if (person is null) return;
        await GreetingState.SetAsync($"Hello {person.FirstName}!", ct);
    }
}
```

MVUX manages the subscription lifetime — it's disposed when the Model is garbage collected.

### Awaiting a State

States can be awaited directly to get the current value:

```csharp
string? currentCity = await this.City;
```

---

## States vs Feeds

| Behavior | IFeed\<T\> | IState\<T\> |
|---|---|---|
| Read data | Yes | Yes |
| Write/update data | No | Yes |
| Two-way binding | No | Yes |
| Caches current value | No (stateless) | Yes |
| Replays value on subscribe | No | Yes |
| Lifecycle owner required | No | Yes (`this`) |

**Rule of thumb**: Use `IFeed<T>` for data displayed to the user. Use `IState<T>` when the user can change it or you need to update it programmatically.

---

## Common Mistakes

- **Forgetting `this`**: `State<string>.Empty(this)` — the `this` parameter is required
- **Not awaiting updates**: `await City.UpdateAsync(...)` — fire-and-forget may lose updates
- **Impure updater**: The `UpdateAsync` delegate may run multiple times — don't mutate external state inside it
- **Using IState when IFeed suffices**: States are heavier (cache value, track lifetime) — only use when mutability is needed

---

## Reference

- [State Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Reactive/state.html)
