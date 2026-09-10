# C# Markup - Binding and Resources

Reference for DataContext setup, expression tree bindings, binding modes, transforms, commands, converters, and resource access.

---

### DataContext with Instance vs Type-Only
**Rule**: Use `.DataContext(new ViewModel(), (page, vm) => ...)` when you create the ViewModel inline. Use `.DataContext<ViewModel>((page, vm) => ...)` when the ViewModel is provided externally (e.g., via dependency injection or navigation).
**Why**: The instance overload sets `DataContext` to the provided object immediately. The type-only overload declares the expected type for binding expressions without assigning an instance - the actual DataContext must arrive later (through DI, navigation, or parent inheritance). Choosing the wrong overload causes either null DataContext at runtime or unnecessary manual construction bypassing DI.
**Example (C#)**:
```csharp
// Instance overload - ViewModel created inline
public sealed partial class MainPage : Page
{
    public MainPage()
    {
        this.DataContext(new MainViewModel(), (page, vm) => page
            .Content(
                new TextBlock().Text(() => vm.Title)
            )
        );
    }
}

// Type-only overload - ViewModel resolved externally (DI, navigation, etc.)
public sealed partial class MainPage : Page
{
    public MainPage()
    {
        this.DataContext<MainViewModel>((page, vm) => page
            .Content(
                new TextBlock().Text(() => vm.Title)
            )
        );
    }
}
```
**Common Mistakes**:
- Using the type-only overload but never setting DataContext from the outside. The page will render with no data.
- Constructing a ViewModel with `new` inside the type-only generic overload. It does not accept an instance argument.
- Nesting DataContext calls without binding the inner DataContext to a property of the parent:
  ```csharp
  // CORRECT - bind inner DataContext to parent property
  new StackPanel()
      .DataContext(() => vm.ChildModel, (panel, child) => panel
          .Children(new TextBlock().Text(() => child.Name))
      )
  ```
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/GettingStarted.html

---

### Expression Tree Binding - The vm Placeholder
**Rule**: Treat `vm` (and all DataContext placeholders) as compile-time expression trees only. Never call methods on `vm`, read its values imperatively, or store it in a variable outside the delegate.
**Why**: The `vm` parameter is always `null` at configuration time. The lambda `() => vm.Property` is captured as an `Expression<Func<T>>` and converted into a WinUI Binding path string (e.g., `"Property"`). Evaluating `vm.Property` directly causes a `NullReferenceException` at construction time.
**Example (C#)**:
```csharp
this.DataContext<MainViewModel>((page, vm) => page
    .Content(
        new StackPanel().Children(
            // CORRECT - expression tree, generates Binding Path="Count"
            new TextBlock().Text(() => vm.Count),

            // CORRECT - nested path, generates Binding Path="User.DisplayName"
            new TextBlock().Text(() => vm.User.DisplayName),

            // CORRECT - command binding, generates Binding Path="SaveCommand"
            new Button().Command(() => vm.SaveCommand)
        )
    )
);
```
**Common Mistakes**:
- Using `vm` imperatively:
  ```csharp
  // WRONG - vm is null, this throws NullReferenceException
  var title = vm.Title;
  new TextBlock().Text(title)
  ```
- Using complex expressions that are not simple property paths:
  ```csharp
  // WRONG - the expression tree must be a simple member access path
  new TextBlock().Text(() => vm.Items.Count.ToString())
  ```
  Use a transform delegate instead: `.Text(() => vm.Items.Count, c => c.ToString())`
- Assigning `vm` to a field or local outside the delegate scope. The placeholder has no meaningful value.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Overview.html#data-binding

---

### Binding Modes - OneWay, TwoWay, OneTime
**Rule**: Use the shorthand `() => vm.Property` for OneWay (default). Use the builder pattern `x => x.Bind(() => vm.Property).TwoWay()` for TwoWay or OneTime bindings.
**Why**: OneWay is the most common mode and gets the concise syntax. TwoWay is required for input controls (`TextBox`, `Slider`, `ToggleSwitch`) where user input must flow back to the ViewModel. OneTime is optimal for static data that never changes, reducing binding overhead.
**Example (C#)**:
```csharp
// OneWay (default) - ViewModel to View only
new TextBlock()
    .Text(() => vm.DisplayName)

// TwoWay - View and ViewModel synchronize
new TextBox()
    .Text(x => x.Bind(() => vm.SearchQuery).TwoWay())

new Slider()
    .Value(x => x.Bind(() => vm.Volume).TwoWay())

new ToggleSwitch()
    .IsOn(x => x.Bind(() => vm.IsEnabled).TwoWay())

// OneTime - set once, never updated
new TextBlock()
    .Text(x => x.Bind(() => vm.AppVersion).OneTime())
```
**Common Mistakes**:
- Using shorthand `() => vm.SearchQuery` on a `TextBox.Text` and wondering why the ViewModel does not update. The shorthand is OneWay only. Use `.Bind(...).TwoWay()`.
- Applying `.TwoWay()` to read-only display controls like `TextBlock`. This compiles but is wasteful since `TextBlock.Text` is never set by the user.
- Forgetting `.TwoWay()` on `CheckBox.IsChecked`, `Slider.Value`, or other input controls. The ViewModel property will remain at its initial value.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Overview.html#data-binding

---

### Binding with Transform Delegate
**Rule**: Use the second parameter of the shorthand binding to transform bound values: `.Text(() => vm.Property, value => transformedString)`. For more complex transforms, use `.Bind(...).Convert(...)`.
**Why**: The transform delegate runs on every binding update, replacing the need for simple `IValueConverter` implementations. This keeps formatting logic co-located with the UI definition and avoids creating single-use converter classes.
**Example (C#)**:
```csharp
// Shorthand transform - appends prefix
new TextBlock()
    .Text(() => vm.Count, count => $"Counter: {count}")

// Shorthand transform - conditional text
new TextBlock()
    .Text(() => vm.ItemCount, count => count == 0 ? "No items" : $"{count} items")

// Builder pattern with Convert - when you need the full builder
new TextBlock()
    .Foreground(x => x.Binding(() => vm.IsValid)
        .Convert(valid => new SolidColorBrush(valid ? Colors.Green : Colors.Red)))

// Builder pattern with Convert and ConvertBack - for TwoWay scenarios
new TextBox()
    .Text(x => x.Binding(() => vm.Amount)
        .Convert(amount => amount.ToString("C2"))
        .ConvertBack(text => decimal.TryParse(text, out var v) ? v : 0m))
```
**Common Mistakes**:
- Using the transform delegate for expensive operations (database queries, network calls). The delegate runs on every property change on the UI thread. Keep it pure and fast.
- Forgetting that the shorthand transform `() => vm.Prop, val => ...` only supports OneWay. For TwoWay with conversion, use the builder with `.Convert()` and `.ConvertBack()`.
- Returning a new object (e.g., `new SolidColorBrush(...)`) on every update in a high-frequency binding. Cache or reuse brushes where possible.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Converters.html

---

### Command Binding
**Rule**: Bind commands using `() => vm.CommandProperty` on the `.Command()` extension. The property must be an `ICommand` (or MVUX implicit `IAsyncCommand`).
**Why**: The expression tree extracts the property path and creates a standard WinUI `{Binding Path=CommandProperty}`. This supports `CanExecute` for automatic button enable/disable behavior, matching XAML command binding semantics exactly.
**Example (C#)**:
```csharp
// Standard ICommand binding
new Button()
    .Content("Save")
    .Command(() => vm.SaveCommand)

// With CommandParameter
new Button()
    .Content("Delete")
    .Command(() => vm.DeleteCommand)
    .CommandParameter(() => vm.SelectedItem)

// MVUX - public methods auto-generate IAsyncCommand properties
// Model has: public async ValueTask Save() { ... }
// Binding uses the implicit command name:
new Button()
    .Content("Save")
    .Command(() => vm.Save)
```
**Common Mistakes**:
- Binding to a method directly instead of a command property. The expression `() => vm.DoSomething()` with parentheses is a method call, not a member access. Remove the parentheses.
- Using `.Command(() => vm.Save)` when the ViewModel exposes `SaveCommand` (CommunityToolkit.Mvvm `[RelayCommand]` generates `SaveCommand` from a `Save()` method).
- Forgetting `CommandParameter` when the command's `Execute` method expects an argument.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Overview.html

---

### ThemeResource.Get and StaticResource.Get
**Rule**: Use `ThemeResource.Get<T>("Key")` for theme-aware resources (colors, brushes) that change with light/dark mode. Use `StaticResource.Get<T>("Key")` for resources that remain constant regardless of theme.
**Why**: `ThemeResource.Get` generates a `{ThemeResource}` markup equivalent that re-evaluates when the app theme changes. `StaticResource.Get` generates a `{StaticResource}` equivalent that resolves once. Using `StaticResource` for theme-dependent values causes stale colors after theme switches.
**Example (C#)**:
```csharp
// ThemeResource - changes with light/dark theme
this.Background(ThemeResource.Get<Brush>("ApplicationPageBackgroundThemeBrush"))

new TextBlock()
    .Foreground(ThemeResource.Get<Brush>("SystemControlForegroundBaseHighBrush"))

// StaticResource - resolved once, does not track theme changes
new TextBlock()
    .Style(StaticResource.Get<Style>("TitleLarge"))

// Creating and adding resources to a page
this.Resources(r => r
    .Add("PageAccent", Colors.DodgerBlue, Colors.CornflowerBlue)  // light, dark
    .Add("FixedSpacing", 16.0)  // theme-independent
)

// Creating typed resources with the static helper
public static class AppResources
{
    public static readonly Resource<Color> AccentColor =
        ThemeResource.Create<Color>(nameof(AccentColor), "#0078D4", "#60CDFF");

    public static readonly Resource<Geometry> ChevronIcon =
        StaticResource.Create<Geometry>(nameof(ChevronIcon), "M 4 0 L 10 6 L 4 12");
}
```
**Common Mistakes**:
- Using `StaticResource.Get` for brushes or colors that must respond to theme changes. The value will be frozen to whichever theme was active at resolution time.
- Misspelling the resource key string. There is no compile-time validation of resource keys. Use `nameof()` or constants for custom resource keys.
- Forgetting to add the resource to a `ResourceDictionary` before referencing it. The resource must exist in the visual tree's resource chain.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Resources.html

---

### Converter Usage with IValueConverter
**Rule**: For reusable value conversions, define `IValueConverter` implementations as static fields and pass them via `.Converter()` in the builder pattern. For one-off conversions, use `.Convert()` inline.
**Why**: `IValueConverter` instances are reusable across the entire app, avoiding duplicate lambda allocations. The `.Convert()` inline delegate is convenient for page-specific formatting but creates a new converter instance per binding. For bindings repeated in templates (e.g., `ItemTemplate` with 100+ items), a shared `IValueConverter` reduces memory allocation.
**Example (C#)**:
```csharp
// Reusable converter - defined once
public static class Converters
{
    public static readonly IValueConverter InverseBool = new InverseBoolConverter();
    public static readonly IValueConverter DateFormat = new DateFormatConverter();
}

// Used via builder pattern
new Button()
    .IsEnabled(x => x.Binding(() => vm.IsBusy)
        .Converter(Converters.InverseBool))

new TextBlock()
    .Text(x => x.Binding(() => vm.CreatedDate)
        .Converter(Converters.DateFormat))

// Inline Convert - for one-off transforms
new TextBlock()
    .Text(x => x.Binding(() => vm.Score)
        .Convert(score => $"{score:P0}"))

// Inline with conditional logic
new Border()
    .Background(x => x.Binding(() => vm.Status)
        .Convert(status => new SolidColorBrush(
            status == "Active" ? Colors.Green : Colors.Gray)))
```
**Common Mistakes**:
- Creating a new converter instance inside the binding expression every time the page is constructed. Define converters as `static readonly` fields.
- Using `.Converter()` and `.Convert()` on the same binding. They serve the same purpose; use one or the other.
- Forgetting that `ConverterParameter` is available on the builder: `.Binding(...).ConverterParameter("format_string")`.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Converters.html
