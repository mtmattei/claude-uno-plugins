# Records and Code Generation

How to design immutable data models for MVUX, configure key equality, and understand the source generator's requirements.

---

## Immutable Records

**Impact**: CRITICAL — MVUX requires immutability for predictable state changes and thread safety.

### Model Records

Model classes must be `partial record` with a `Model` suffix. The source generator produces a corresponding ViewModel class.

```csharp
// CORRECT: partial record with Model suffix
public partial record MainModel(IWeatherService WeatherService)
{
    public IFeed<WeatherInfo> CurrentWeather =>
        Feed.Async(WeatherService.GetCurrentWeather);
}
// Generator produces: MainViewModel
```

```csharp
// WRONG: missing partial — generator cannot produce ViewModel
public record MainModel(IWeatherService WeatherService) { ... }

// WRONG: missing Model suffix — generator ignores this class
public partial record MainPresenter(IWeatherService WeatherService) { ... }

// WRONG: class instead of record — loses immutability guarantees
public partial class MainModel { ... }
```

### Customizing the Model Name Pattern

The default regex is `Model$`. You can change it:

```csharp
// Per-class: opt-in any record
[ReactiveBindable]
public partial record MainPresenter(IWeatherService WeatherService) { ... }

// Per-assembly: change the pattern for all classes
[assembly: ImplicitBindables(@"MyApp\.Presentation\..*Model$")]
```

### Data Entity Records

Data entities (the objects returned by services) should also be `record` types:

```csharp
// CORRECT: primary constructor — immutable by default
public partial record Person(int Id, string FirstName, string LastName);

// CORRECT: init-only properties
public partial record Person
{
    public int Id { get; init; }
    public string FirstName { get; init; }
    public string LastName { get; init; }
}

// CORRECT: mixed approach
public partial record Person(int Id, string FirstName)
{
    public string LastName { get; init; } = string.Empty;
}
```

```csharp
// WRONG: set instead of init — breaks immutability
public partial record Person
{
    public int Id { get; set; }  // Mutable!
    public string FirstName { get; set; }  // Mutable!
}
```

### Updating Records with `with` Expressions

Since records are immutable, create modified copies using `with`:

```csharp
var updated = person with { FirstName = "Jane" };

// Inside a state update:
await PersonState.UpdateAsync(current => current with { FirstName = "Jane" }, ct);
```

---

## Key Equality

**Impact**: HIGH — Required for messaging (entity matching) and selection tracking in collections.

### Automatic Generation

`IKeyEquatable<T>` is automatically generated for any `partial record` that has a property named `Id` or `Key`:

```csharp
// Auto-generates IKeyEquatable<Person> using Id
public partial record Person(int Id, string FirstName, string LastName);

// Auto-generates IKeyEquatable<Setting> using Key
public partial record Setting(string Key, string Value);
```

### Custom Key Configuration

```csharp
// Explicit key with [Key] attribute (overrides implicit detection)
public partial record MyItem(
    [property: Key] Guid EntityId,
    [property: Key] string SourceId,
    string Value);

// Using [ImplicitKeyEquality] attribute
[ImplicitKeyEquality("EntityId")]
public partial record MyItem(Guid EntityId, string SourceId, string Value);

// Assembly-wide default key names
[assembly: ImplicitKeyEquality("Id", "Key", "EntityId")]
```

### Disabling Key Equality Generation

```csharp
// Per-type
[ImplicitKeys(IsEnabled = false)]
public partial record MyItem(string Value);

// Per-assembly
[assembly: ImplicitKeys(IsEnabled = false)]
```

### How Key Equality Works

```csharp
var john1 = new Person(1, "John", "Doe");
var john2 = john1 with { LastName = "Smith" };

john1.Equals(john2);     // false — different LastName
john1.KeyEquals(john2);   // true — same Id

// This matters for ListState: when messaging sends an Updated entity,
// the list finds the matching item by key and replaces it.
```

### Additional Context

- Only ONE implicit key property is used (first match from the configured list)
- Records with `[Key]` attributes bypass implicit key detection entirely
- Both `Uno.Extensions.Equality.KeyAttribute` and `System.ComponentModel.DataAnnotations.KeyAttribute` work
- If you have both `Id` and `Key` properties, only `Id` is used (it's checked first)

---

## Service Return Types

**Impact**: MEDIUM — Correct return types prevent runtime issues.

Services consumed by MVUX models should return:

| Feed Type | Service Return Type |
|---|---|
| `IFeed<T>` | `ValueTask<T>` or `Task<T>` |
| `IListFeed<T>` | `ValueTask<IImmutableList<T>>` or `Task<IImmutableList<T>>` |
| Streaming | `IAsyncEnumerable<T>` |

```csharp
// CORRECT: service interface
public interface IWeatherService
{
    ValueTask<WeatherInfo> GetCurrentWeather(CancellationToken ct);
    ValueTask<IImmutableList<City>> GetCities(CancellationToken ct);
}
```

Always include `CancellationToken` — it enables cancellation when the ViewModel is disposed.

---

## Reference

- [MVUX Overview: Records](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/WorkingWithRecords.html)
- [Key Equality Concept](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/KeyEquality/concept.html)
