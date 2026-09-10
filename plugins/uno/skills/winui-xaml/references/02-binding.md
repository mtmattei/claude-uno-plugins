# Binding Skills - WinUI 3 XAML

## BINDING-001: Prefer x:Bind Over Binding

**Rule**: Use `{x:Bind}` instead of `{Binding}` for all data binding scenarios.

**Why**: x:Bind is compiled at build time, providing type safety, compile-time error detection, and 3-4x better performance. Binding uses runtime reflection which is slower and catches errors only at runtime.

**Example (Correct)**:
```xml
<Page x:Class="MyApp.MainPage">
    <StackPanel>
        <TextBlock Text="{x:Bind ViewModel.Title, Mode=OneWay}"/>
        <TextBlock Text="{x:Bind ViewModel.Subtitle}"/>
        <Button Command="{x:Bind ViewModel.SaveCommand}"/>
    </StackPanel>
</Page>
```

**Common Mistakes**:
```xml
<!-- WRONG: Runtime Binding - slower, no compile-time checking -->
<TextBlock Text="{Binding Title}"/>
<TextBlock Text="{Binding Path=User.Name}"/>
```

**Performance Comparison**:
| Metric | {Binding} | {x:Bind} |
|--------|-----------|----------|
| Initial bind (1000 items) | ~45ms | ~12ms |
| Memory overhead | ~1.2MB | ~0.4MB |
| Update time | ~8ms | ~3ms |

**Key Differences**:
- x:Bind defaults to `Mode=OneTime` (Binding defaults to `OneWay`)
- x:Bind requires explicit `Mode=OneWay` or `Mode=TwoWay` for updates
- x:Bind supports function binding
- x:Bind catches binding path errors at compile time

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/xaml-platform/x-bind-markup-extension

---

## BINDING-002: Use Appropriate Binding Modes

**Rule**: Always specify the minimal binding mode required. Use OneTime for static data, OneWay for read-only dynamic data, TwoWay only for editable fields.

**Why**: Each binding mode has different overhead. OneTime has zero update cost. OneWay tracks source changes. TwoWay tracks both directions. Unnecessary tracking wastes CPU cycles.

**Example (Correct)**:
```xml
<!-- Static data that never changes -->
<TextBlock Text="{x:Bind User.Id}"/>  <!-- OneTime is default for x:Bind -->

<!-- Data that updates from source -->
<TextBlock Text="{x:Bind User.Status, Mode=OneWay}"/>

<!-- User-editable field -->
<TextBox Text="{x:Bind User.Name, Mode=TwoWay}"/>
<CheckBox IsChecked="{x:Bind User.IsActive, Mode=TwoWay}"/>
```

**Decision Tree**:
```
Does the data NEVER change after initial load?
 -> YES: Mode=OneTime (or omit for x:Bind)

Does the user edit this value?
 -> YES: Mode=TwoWay
 -> NO: Does the source property change?
        -> YES: Mode=OneWay
        -> NO: Mode=OneTime
```

**Common Mistakes**:
```xml
<!-- WRONG: TwoWay on read-only display -->
<TextBlock Text="{x:Bind User.Name, Mode=TwoWay}"/>

<!-- WRONG: OneWay when data is static -->
<TextBlock Text="{x:Bind AppVersion, Mode=OneWay}"/>
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/data-binding/data-binding-in-depth#binding-modes

---

## BINDING-003: Use Function Binding for Transformations

**Rule**: Use x:Bind function binding instead of IValueConverter for data transformations.

**Why**: Function binding is compiled, type-safe, and avoids the boxing/unboxing overhead of IValueConverter. It's also easier to debug and test. Impact: 5x faster than converters.

**Example (Correct)**:
```xml
<Page
    xmlns:local="using:MyApp"
    xmlns:helpers="using:MyApp.Helpers">

    <!-- Direct function call -->
    <TextBlock Text="{x:Bind helpers:Formatters.FormatPrice(Product.Price)}"/>

    <!-- Multiple parameters -->
    <TextBlock Text="{x:Bind helpers:Formatters.FormatFullName(User.FirstName, User.LastName)}"/>

    <!-- Boolean to Visibility -->
    <Grid Visibility="{x:Bind helpers:Converters.BoolToVisibility(IsVisible), Mode=OneWay}"/>
</Page>
```

```csharp
// Helpers/Formatters.cs
public static class Formatters
{
    public static string FormatPrice(decimal price) => $"${price:F2}";

    public static string FormatFullName(string first, string last) => $"{first} {last}";
}

public static class Converters
{
    public static Visibility BoolToVisibility(bool value)
        => value ? Visibility.Visible : Visibility.Collapsed;
}
```

**Common Mistakes**:
```xml
<!-- WRONG: IValueConverter has boxing overhead -->
<TextBlock Text="{x:Bind Product.Price, Converter={StaticResource PriceConverter}}"/>
```

```csharp
// WRONG: Converter allocates and boxes on every call
public class PriceConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, string language)
    {
        return $"${(decimal)value:F2}"; // Boxing/unboxing
    }
    // ...
}
```

**Uno Platform Notes**: x:Bind function binding is fully supported. Ensure functions are public static or instance methods on the page/control.

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/data-binding/function-bindings

---

## BINDING-004: Avoid Binding in Hot Paths

**Rule**: For frequently updated values (animations, real-time data), consider direct property assignment over binding.

**Why**: Binding has overhead even with x:Bind. For values updating 60+ times per second, direct assignment avoids the binding evaluation cost.

**Example (Correct)**:
```csharp
// Direct assignment for high-frequency updates
private void OnTimerTick(object sender, object e)
{
    // Update directly instead of through bound property
    ClockText.Text = DateTime.Now.ToString("HH:mm:ss.fff");
    ProgressIndicator.Value = _currentProgress;
}
```

```xml
<!-- Named elements for direct access -->
<TextBlock x:Name="ClockText"/>
<ProgressBar x:Name="ProgressIndicator" Maximum="100"/>
```

**Common Mistakes**:
```csharp
// WRONG: Binding overhead for 60fps updates
public string CurrentTime
{
    get => _currentTime;
    set => SetProperty(ref _currentTime, value); // INotifyPropertyChanged fires
}

private void OnTimerTick(object sender, object e)
{
    CurrentTime = DateTime.Now.ToString("HH:mm:ss.fff"); // Binding evaluates
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-xaml-layout

---

## BINDING-005: Batch Property Change Notifications

**Rule**: When updating multiple related properties, batch notifications to reduce binding re-evaluation.

**Why**: Each PropertyChanged notification can trigger multiple binding updates and UI refreshes. Batching reduces the total number of updates.

**Example (Correct)**:
```csharp
public void UpdateUser(UserDto dto)
{
    // Set backing fields without notification
    _firstName = dto.FirstName;
    _lastName = dto.LastName;
    _email = dto.Email;

    // Single batched notification
    OnPropertyChanged(string.Empty); // Empty = all properties changed

    // Or notify specific properties
    OnPropertyChanged(nameof(FirstName));
    OnPropertyChanged(nameof(LastName));
    OnPropertyChanged(nameof(Email));
}
```

**With CommunityToolkit.Mvvm**:
```csharp
using CommunityToolkit.Mvvm.ComponentModel;

public partial class UserViewModel : ObservableObject
{
    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(FullName))]
    private string _firstName;

    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(FullName))]
    private string _lastName;

    public string FullName => $"{FirstName} {LastName}";
}
```

**Common Mistakes**:
```csharp
// WRONG: Three separate notification cycles
public void UpdateUser(UserDto dto)
{
    FirstName = dto.FirstName; // Fires PropertyChanged
    LastName = dto.LastName;   // Fires PropertyChanged
    Email = dto.Email;         // Fires PropertyChanged
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.inotifypropertychanged

---

## BINDING-006: Use UpdateSourceTrigger Appropriately

**Rule**: For text input with TwoWay binding, consider using explicit update triggers or debouncing instead of PropertyChanged.

**Why**: Default PropertyChanged trigger fires on every keystroke, which can cause excessive validation/search operations. Use LostFocus or debouncing for expensive operations.

**Example (Correct)**:
```xml
<!-- Update only when user leaves field -->
<TextBox Text="{x:Bind ViewModel.SearchQuery, Mode=TwoWay, UpdateSourceTrigger=LostFocus}"/>
```

```csharp
// Or implement debouncing for search-as-you-type
private DispatcherQueueTimer _debounceTimer;

private void SearchBox_TextChanged(object sender, TextChangedEventArgs e)
{
    _debounceTimer?.Stop();
    _debounceTimer = DispatcherQueue.CreateTimer();
    _debounceTimer.Interval = TimeSpan.FromMilliseconds(300);
    _debounceTimer.Tick += (s, args) =>
    {
        _debounceTimer.Stop();
        ViewModel.SearchQuery = SearchBox.Text;
    };
    _debounceTimer.Start();
}
```

**Common Mistakes**:
```xml
<!-- WRONG: Fires expensive search on every keystroke -->
<TextBox Text="{x:Bind ViewModel.SearchQuery, Mode=TwoWay}"/>
```

```csharp
// WRONG: SearchQuery setter triggers API call immediately
public string SearchQuery
{
    get => _searchQuery;
    set
    {
        _searchQuery = value;
        OnPropertyChanged();
        _ = PerformSearchAsync(); // Called on every keystroke!
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/data-binding/data-binding-in-depth

---

## BINDING-007: Avoid StringFormat in Bindings

**Rule**: In WinUI 3 and Uno Platform, do not use `{Binding StringFormat=...}`. Use x:Bind function binding or Run elements for string concatenation.

**Why**: StringFormat is a WPF feature that doesn't work reliably in WinUI 3. Use x:Bind functions or multiple Run elements for formatted text.

**Example (Correct)**:
```xml
<!-- Function binding for formatting -->
<TextBlock Text="{x:Bind local:Formatters.FormatCurrency(Price), Mode=OneWay}"/>

<!-- Multiple Run elements for concatenation -->
<TextBlock>
    <Run Text="Total: "/>
    <Run Text="{x:Bind Total, Mode=OneWay}"/>
    <Run Text=" items"/>
</TextBlock>
```

**Common Mistakes**:
```xml
<!-- WRONG: StringFormat is WPF-only -->
<TextBlock Text="{Binding Price, StringFormat='${0:F2}'}"/>

<!-- WRONG: x:Static is WPF-only -->
<TextBlock Text="{x:Static local:Constants.AppName}"/>
```

**Uno Platform Notes**: Uno Platform follows WinUI 3 behavior. StringFormat is not supported. Use function binding or Run concatenation.

**Reference**: https://platform.uno/docs/articles/features/windows-ui-xaml-xbind.html
