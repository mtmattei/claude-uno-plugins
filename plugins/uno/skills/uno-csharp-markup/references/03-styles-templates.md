# C# Markup - Styles, Templates, and Visual States

Reference for Style<T> builder, style inheritance, DataTemplate, ItemTemplate, VisualStateManager, AdaptiveTrigger, and attached properties.

---

### Style<T> Builder and Setters
**Rule**: Create styles using `new Style<T>().Setters(s => s.Property(value))`. The lambda parameter `s` provides the same fluent extensions available on the control itself.
**Why**: `Style<T>` is generic, so the Setters lambda is strongly typed to the target control. This means IntelliSense shows only valid properties for that control type, catching errors at compile time. In XAML, a misspelled property name in a Setter is a silent runtime failure.
**Example (C#)**:
```csharp
// Implicit style for all TextBlocks in scope
new Style<TextBlock>()
    .Setters(s => s
        .FontSize(14)
        .Foreground(ThemeResource.Get<Brush>("SystemControlForegroundBaseHighBrush"))
        .TextWrapping(TextWrapping.Wrap)
    )

// Named style added to resources
this.Resources(r => r
    .Add("HeaderStyle", new Style<TextBlock>()
        .Setters(s => s
            .FontSize(28)
            .FontWeight(FontWeights.Bold)
            .Margin(0, 0, 0, 12)
        )
    )
)

// Apply a named style to a control
new TextBlock()
    .Style(StaticResource.Get<Style>("HeaderStyle"))
    .Text("Page Title")

// Implicit style (no key) - applies to all TextBlocks in the resource scope
this.Resources(r => r
    .Add(new Style<TextBlock>()
        .Setters(s => s.FontSize(14))
    )
)
```
**Common Mistakes**:
- Using a non-generic `Style` instead of `Style<T>`. The non-generic version loses type safety and IntelliSense for setters.
- Setting properties both in a style and inline on the control. Inline values override style setters, which can cause confusion when debugging visual inconsistencies.
- Forgetting to add the style to a `ResourceDictionary`. A style object that is not registered anywhere has no effect unless applied directly via `.Style(myStyleInstance)`.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Styles.html

---

### BasedOn for Style Inheritance
**Rule**: Use `.BasedOn("StyleKey")` to inherit from a globally registered style by key, or `.BasedOn(styleInstance)` to inherit from a local style reference. Use `.Assign(out var)` to capture a style instance for reuse.
**Why**: Style inheritance avoids duplicating setters across related styles. A "SubtitleStyle" can inherit from "BodyStyle" and only override font size, keeping the design system DRY. Changes to the base style propagate automatically to derived styles.
**Example (C#)**:
```csharp
// BasedOn a global resource key (e.g., Material theme style)
var cardTitleStyle = new Style<TextBlock>()
    .BasedOn("TitleMedium")
    .Setters(s => s
        .Foreground(ThemeResource.Get<Brush>("OnSurfaceBrush"))
        .Margin(0, 0, 0, 4)
    );

// BasedOn a local instance using .Assign()
new Style<TextBlock>()
    .Setters(s => s
        .FontSize(16)
        .TextWrapping(TextWrapping.Wrap)
    )
    .Assign(out var baseTextStyle);

var emphasisTextStyle = new Style<TextBlock>()
    .BasedOn(baseTextStyle)
    .Setters(s => s
        .FontWeight(FontWeights.SemiBold)
    );

// Register both in resources
this.Resources(r => r
    .Add("BaseText", baseTextStyle)
    .Add("EmphasisText", emphasisTextStyle)
);
```
**Common Mistakes**:
- Using `.BasedOn("StyleKey")` for a style that is not globally registered (e.g., only added to a page-level ResourceDictionary). The key-based overload resolves from `Application.Resources` only. Use the instance-based overload for non-global styles.
- Chaining `.BasedOn()` after `.Setters()`. Order does not matter for the resulting style, but placing `.BasedOn()` first improves readability by showing the inheritance chain before overrides.
- Inheriting across different control types (e.g., a `Style<Button>` based on a `Style<TextBlock>`). WinUI requires the target type to match or be a subclass.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Styles.html

---

### DataTemplate and ItemTemplate
**Rule**: Use `.ItemTemplate<TItem>(item => ...)` for typed item templates in `ListView`, `ItemsRepeater`, and similar controls. The `item` parameter is an expression tree placeholder (like `vm`), not a live object.
**Why**: The generic `ItemTemplate<T>` provides a strongly typed lambda where the item placeholder enables binding expressions against the item type. This replaces XAML's `DataTemplate` with compile-time validation of property paths on the item model.
**Example (C#)**:
```csharp
// Typed ItemTemplate - item is a placeholder for binding
new ListView()
    .ItemsSource(() => vm.People)
    .ItemTemplate<PersonModel>(item =>
        new StackPanel()
            .Orientation(Orientation.Horizontal)
            .Spacing(8)
            .Padding(12)
            .Children(
                new PersonPicture()
                    .Width(40).Height(40)
                    .DisplayName(() => item.FullName),
                new StackPanel()
                    .VerticalAlignment(VerticalAlignment.Center)
                    .Children(
                        new TextBlock()
                            .Text(() => item.FullName),
                        new TextBlock()
                            .Text(() => item.Email)
                            .Opacity(0.6)
                    )
            )
    )

// Simple string list - bind directly
new ListView()
    .ItemsSource(() => vm.Tags)
    .ItemTemplate<string>(tag =>
        new TextBlock().Text(() => tag)
    )

// Non-generic ItemTemplate with Bind()
new ListView()
    .ItemsSource(() => vm.SearchResults)
    .ItemTemplate(() => new TextBlock().Text(x => x.Bind()))
```
**Common Mistakes**:
- Treating the `item` parameter as a live reference and calling methods on it:
  ```csharp
  // WRONG - item is null, same as vm in DataContext
  .ItemTemplate<PersonModel>(item => new TextBlock().Text(item.FullName))
  ```
  Always use an expression tree: `.Text(() => item.FullName)`.
- Forgetting the generic type argument: `.ItemTemplate(item => ...)` without `<PersonModel>` loses type safety and IntelliSense on the item's properties.
- Nesting a `ListView` inside an `ItemTemplate` without constraining its height, causing layout performance issues.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/HowTo-UseDataTemplates.html

---

### Control Templates in Styles
**Rule**: Define control templates inside a `Style<T>` using `.Template(b => ...)` within the `.Setters(...)` block. Use `x => x.TemplateBind(() => b.Property)` for template bindings.
**Why**: Control templates allow full visual customization of a control's appearance. The `TemplateBind` expression maps to `{TemplateBinding}` in XAML, binding the template's visual elements to the templated control's properties.
**Example (C#)**:
```csharp
new Style<Button>()
    .Setters(setters => setters
        .Template(b => new Border()
            .Background(x => x.TemplateBind(() => b.Background))
            .CornerRadius(8)
            .Padding(16, 8)
            .Child(
                new ContentPresenter()
                    .Content(x => x.TemplateBind(() => b.Content))
                    .HorizontalAlignment(HorizontalAlignment.Center)
                    .VerticalAlignment(VerticalAlignment.Center)
            )
        )
    )
```
**Common Mistakes**:
- Using regular `Bind(() => vm.Property)` instead of `TemplateBind(() => b.Property)` inside a control template. Regular bindings look at `DataContext`; template bindings look at the templated parent.
- Forgetting the `ContentPresenter` in a `Button` template. Without it, the button's `Content` property has nowhere to render.
- Omitting visual states from a custom template. The control loses all interactive feedback (hover, pressed, disabled) unless VisualStateManager groups are added.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Styles.html#templates

---

### VisualStateManager in C# Markup
**Rule**: Add visual state groups to the root panel of a page or UserControl using the `VisualStateManager` fluent API. Define states with property setters that activate under specific conditions.
**Why**: VisualStateManager drives responsive layouts, interactive states, and animations in WinUI. In C# Markup, it follows the same model as XAML but with fluent builder syntax. Without it, responsive design and control state feedback require manual code-behind event handling.
**Example (C#)**:
```csharp
new Grid()
    .RowDefinitions("Auto, *")
    .VisualStateManager(vsm => vsm
        .Group(g => g.Name("WindowSizeStates")
            .State(s => s.Name("Narrow")
                .StateTriggers(
                    new AdaptiveTrigger().MinWindowWidth(0)
                )
            )
            .State(s => s.Name("Wide")
                .StateTriggers(
                    new AdaptiveTrigger().MinWindowWidth(720)
                )
            )
        )
    )
    .Children(
        new TextBlock().Text("Header").Grid(row: 0),
        new StackPanel().Name("SidePanel").Grid(row: 1)
    )
```
**Common Mistakes**:
- Placing `VisualStateManager` on a child element instead of the root container of the page or UserControl. VSM groups must be on the root visual element.
- Forgetting to set `.Name("ElementName")` on target elements. The `Setter.Target` path requires the element to be named.
- Defining states without `StateTriggers`. The states will never activate unless triggered programmatically via `VisualStateManager.GoToState()`.
**Uno Platform Notes**: The `{Responsive}` markup extension from Uno Toolkit is not directly available in C# Markup. Use `AdaptiveTrigger` with `VisualStateManager` as the alternative for responsive layout breakpoints.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/HowTo-CustomMarkupProject-VisualStates.html

---

### Attached Properties Syntax
**Rule**: Set attached properties using the extension method named after the owning type. Use named parameters for clarity (e.g., `.Grid(row: 1, column: 2)`) or the builder pattern for binding support (e.g., `.Grid(g => g.Row(1).Column(2))`).
**Why**: Attached properties in WinUI are static methods on a declaring type (e.g., `Grid.SetRow`). C# Markup generates friendly extensions that group related attached properties. Named parameters prevent position-based mistakes in multi-parameter calls.
**Example (C#)**:
```csharp
// Shorthand with named parameters
new TextBlock()
    .Text("Cell Content")
    .Grid(row: 2, column: 1, columnSpan: 2)

// Builder pattern - needed for binding or resource references
new TextBlock()
    .Grid(g => g
        .Row(2)
        .Column(1)
        .ColumnSpan(2))

// AutomationProperties - builder for binding support
new TextBlock()
    .AutomationProperties(x => x
        .Name(() => vm.AccessibleLabel))

// Canvas attached properties
new Rectangle()
    .Width(50).Height(50)
    .Canvas(left: 100, top: 200)

// ScrollViewer attached properties
new ItemsRepeater()
    .ScrollViewer(sv => sv
        .HorizontalScrollBarVisibility(ScrollBarVisibility.Auto)
        .VerticalScrollBarVisibility(ScrollBarVisibility.Auto))
```
**Common Mistakes**:
- Writing `Grid.SetRow(element, 2)` imperatively. Use the fluent `.Grid(row: 2)` extension instead to maintain the declarative chain.
- Using positional arguments without names: `.Grid(2, 1)`. This compiles but is ambiguous to readers. Always use `row:` and `column:` parameter names.
- Trying to use a fluent extension for an attached property that has no generated extension. Fall back to the builder pattern with `.Add(DependencyProperty, value)`.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/AttachedProperties.html
