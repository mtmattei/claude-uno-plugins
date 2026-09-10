# Feeds and FeedView

How to create `IFeed<T>` for read-only async data, display it with `FeedView`, and transform it with operators.

---

## Creating Feeds

**Impact**: HIGH — Feeds are the primary way to load and expose async data in MVUX.

### Feed.Async — From a Task-Returning Method

The most common factory. Wraps a `Task<T>` or `ValueTask<T>` method:

```csharp
public partial record MainModel(IWeatherService WeatherService)
{
    // Method reference (preferred — clean and concise)
    public IFeed<WeatherInfo> CurrentWeather =>
        Feed.Async(WeatherService.GetCurrentWeather);

    // Lambda (when you need to pass parameters)
    public IFeed<WeatherInfo> Weather =>
        Feed.Async(async ct => await WeatherService.GetCurrentWeather(ct));
}
```

### Feed.AsyncEnumerable — From a Streaming Source

For data that updates over time (polling, real-time):

```csharp
public IFeed<WeatherInfo> LiveWeather =>
    Feed.AsyncEnumerable(() => GetWeatherStream());

private async IAsyncEnumerable<WeatherInfo> GetWeatherStream(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    while (!ct.IsCancellationRequested)
    {
        yield return await _weatherService.GetCurrentWeather(ct);
        await Task.Delay(TimeSpan.FromMinutes(5), ct);
    }
}
```

**Important**: Always use `[EnumeratorCancellation]` on the `CancellationToken` parameter. The token is cancelled when the last subscription is removed (typically when the ViewModel is disposed).

### Feed.Create — Advanced (Message-Level Control)

For direct control over messages. Rarely needed in app code:

```csharp
public IFeed<WeatherInfo> Weather => Feed.Create(GetWeather);

private async IAsyncEnumerable<Message<WeatherInfo>> GetWeather(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    var message = Message<WeatherInfo>.Initial;
    while (!ct.IsCancellationRequested)
    {
        try
        {
            var weather = await _weatherService.GetCurrentWeather(ct);
            yield return message = message.With().Data(weather).Error(default);
        }
        catch (Exception ex)
        {
            yield return message = message.With().Error(ex);
        }
        await Task.Delay(TimeSpan.FromHours(1), ct);
    }
}
```

---

## Feed Operators

**Impact**: MEDIUM — Transform feed data without manual subscriptions.

### Select — Synchronous Projection

```csharp
public IFeed<string> TemperatureText =>
    CurrentWeather.Select(w => $"{w.Temperature}°C");
```

### SelectAsync — Asynchronous Projection

```csharp
public IFeed<string> AlertDetails =>
    Alert.SelectAsync(async (alert, ct) =>
        await _weatherService.GetAlertDetails(alert, ct));
```

### Where — Filtering

If the predicate returns `false`, the feed publishes a `None` message (not empty — it's treated as "no data"):

```csharp
public IFeed<WeatherAlert> ActiveAlert =>
    CurrentWeather
        .Where(weather => weather.Alert is not null)
        .Select(weather => weather.Alert!);
```

### LINQ Query Syntax

Feeds support LINQ:

```csharp
public IFeed<string> Value => from value in _values
                              where value == 42
                              select value.ToString();
```

### Combining State + Feed with SelectAsync

A feed can depend on a state — it auto-refreshes when the state changes:

```csharp
public IState<string> City => State<string>.Empty(this);

// Refreshes automatically whenever City changes
public IFeed<WeatherInfo> CityWeather =>
    City.SelectAsync(WeatherService.GetWeatherForCity);
```

---

## FeedView Control

**Impact**: HIGH — The primary way to render feed data in XAML.

### XAML Namespace

```xml
xmlns:mvux="using:Uno.Extensions.Reactive.UI"
```

### Basic Usage

The default `<DataTemplate>` child acts as the `ValueTemplate`:

```xml
<mvux:FeedView Source="{Binding CurrentWeather}">
    <DataTemplate>
        <TextBlock Text="{Binding Data.Temperature}" />
    </DataTemplate>
</mvux:FeedView>
```

**Key**: Inside templates, `Data` is a property of `FeedViewState` that contains the actual feed value.

### All Templates

| Template | Shown When |
|---|---|
| `ValueTemplate` | Feed has data (default content child) |
| `ProgressTemplate` | Async operation is loading (default: `ProgressRing`) |
| `ErrorTemplate` | Async operation threw an `Exception` |
| `NoneTemplate` | Result is `null` or empty collection |
| `UndefinedTemplate` | Before the async operation is first invoked (brief flash) |

### Refresh Command

`FeedView` exposes a `Refresh` command on `FeedViewState`:

> **`{Binding Refresh}` works in `ValueTemplate` only.** In `ErrorTemplate` the DataContext
> is the **thrown `Exception`**, not `FeedViewState`, so `{Binding Refresh}` silently
> resolves to null and leaves a Retry button that looks enabled and does nothing (`Parent`
> is unavailable there for the same reason). In `ErrorTemplate` you must bind by
> `ElementName` to the `FeedView` — that resolves even from inside a nested
> `ContentTemplate`. Validated ComponentStatesLab, 2026-08-17, Uno.Sdk 6.8.0-dev.12: the
> button's DataContext serialised as `System.InvalidOperationException` with zero `Refresh`
> members, and switching to `ElementName` made the service run again and the component
> recover.

```xml
<!-- Inside ValueTemplate -->
<Button Content="Refresh" Command="{Binding Refresh}" />

<!-- Inside ErrorTemplate, or outside the FeedView entirely: use ElementName -->
<mvux:FeedView x:Name="weatherFeed" Source="{Binding CurrentWeather}">
    <DataTemplate>
        <TextBlock Text="{Binding Data.Temperature}" />
    </DataTemplate>
</mvux:FeedView>
<Button Content="Refresh" Command="{Binding Refresh, ElementName=weatherFeed}" />
```

### RefreshContainer (Pull to Refresh)

```xml
<mvux:FeedView Source="{Binding Products}" RefreshingState="None">
    <DataTemplate>
        <RefreshContainer mvux:RefreshContainerExtensions.Command="{Binding Refresh}">
            <ListView ItemsSource="{Binding Data}" />
        </RefreshContainer>
    </DataTemplate>
</mvux:FeedView>
```

Set `RefreshingState="None"` to avoid double loading indicators.

### FeedViewState Properties

| Property | Type | Description |
|---|---|---|
| `Data` | `T` | Current feed value |
| `Refresh` | `ICommand` | Triggers feed reload |
| `Progress` | `bool` | Whether loading is in progress |
| `Error` | `Exception?` | Error from last operation |
| `Parent` | `object` | Parent DataContext |

---

## Feeds Are Stateless

Feeds do NOT cache values. Each subscription re-invokes the async source. If you need caching or mutation, use `IState<T>` instead.

| Need | Use |
|---|---|
| Display service data, read-only | `IFeed<T>` |
| User input, two-way binding | `IState<T>` |
| Cache value across navigations | `IState<T>` |

---

## Reference

- [Feed Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Reactive/feed.html)
- [FeedView Control](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/FeedView.html)
- [Simple Feed How-To](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Walkthrough/SimpleFeed.howto.html)
