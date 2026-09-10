# C# Markup - Getting Started

Reference for project setup, page structure, control creation, fluent chaining, type helpers, and content patterns.

---

### Project Setup with CSharpMarkup
**Rule**: Enable C# Markup through the `CSharpMarkup` UnoFeature or the `dotnet new unoapp -markup csharp` template.
**Why**: The `CSharpMarkup` UnoFeature adds all required NuGet packages (`Uno.WinUI.Markup`, generators, and supporting libraries) in a single declaration. Manually adding individual packages leads to version mismatches and missing extension methods.
**Example (C#)**:
```xml
<!-- In .csproj -->
<UnoFeatures>
  CSharpMarkup;
  Material;
  Toolkit;
  MVUX;
</UnoFeatures>
```
Or create a new project from the CLI:
```bash
dotnet new unoapp -preset recommended -markup csharp -presentation mvux -o MyApp
```
**Common Mistakes**:
- Adding `Uno.WinUI.Markup` NuGet manually without also adding the generator package. Use `UnoFeatures` instead for correct package resolution.
- Forgetting to add companion Markup packages when using Toolkit or Themes (`Uno.Toolkit.WinUI.Markup`, `Uno.Themes.WinUI.Markup`). These provide the fluent extensions for those libraries' controls and styles.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/GettingStarted.html

---

### Page Structure - .cs Files Only
**Rule**: Define each page as a single `.cs` file inheriting from `Page`. Place all UI construction in the constructor. Do not create `.xaml` / `.xaml.cs` pairs.
**Why**: C# Markup pages are pure C# classes. The fluent API replaces the XAML parser entirely, giving you compile-time validation, full IntelliSense, and refactoring support. Mixing XAML files with C# Markup pages in the same view causes confusion and provides no benefit.
**Example (C#)**:
```csharp
public sealed partial class MainPage : Page
{
    public MainPage()
    {
        this
            .Background(ThemeResource.Get<Brush>("ApplicationPageBackgroundThemeBrush"))
            .Content(
                new StackPanel()
                    .VerticalAlignment(VerticalAlignment.Center)
                    .HorizontalAlignment(HorizontalAlignment.Center)
                    .Children(
                        new TextBlock()
                            .Text("Hello Uno Platform!")
                    )
            );
    }
}
```
**Common Mistakes**:
- Creating a `.xaml` file alongside the C# Markup page. This results in duplicate `InitializeComponent` calls and build errors.
- Omitting the `partial` keyword. The class must remain `partial` because source generators may augment it.
- Setting properties after `this.Content(...)` returns. The fluent chain sets properties on `this` (the Page), so `.Background(...)` must be chained on `this`, not on the content panel.
**Uno Platform Notes**: Hot Design currently supports XAML only, not C# Markup pages. For runtime visual editing, XAML pages are required.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Overview.html

---

### Control Creation and Fluent Chaining
**Rule**: Instantiate controls with `new ControlType()` and chain property setters using the generated fluent extension methods. Each extension method returns the same control instance for further chaining.
**Why**: The fluent pattern produces a declarative tree that mirrors the visual hierarchy. Each extension method is strongly typed against the target DependencyProperty, catching type mismatches at compile time rather than at runtime (unlike XAML string-based attributes).
**Example (C#)**:
```csharp
new TextBlock()
    .Margin(12)
    .TextAlignment(TextAlignment.Center)
    .HorizontalAlignment(HorizontalAlignment.Center)
    .Text("Counter: 0")
```
For properties that accept enums, use the enum value directly:
```csharp
new Button()
    .HorizontalAlignment(HorizontalAlignment.Stretch) // Strongly typed enum
    .VerticalAlignment(VerticalAlignment.Center)
    .Content("Click Me")
```
**Common Mistakes**:
- Using raw property assignment (`textBlock.Text = "Hello"`) instead of fluent extensions. This breaks the declarative chain and loses the readability benefit. Reserve imperative assignment for rare runtime mutations.
- Assigning a property that does not have a generated extension by guessing the method name. If a property has no fluent extension, use the builder pattern or set the property directly after construction.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Overview.html

---

### Automatic Type Conversion
**Rule**: Rely on built-in type helpers for `Thickness`, `Brush`, `Color`, `CornerRadius`, `FontFamily`, `Geometry`, and `ImageSource`. Pass simplified values instead of constructing wrapper types manually.
**Why**: C# Markup provides implicit conversions that match XAML's string-to-type behavior. Using them reduces verbosity by 50-70% for layout properties and avoids unnecessary allocations from explicit constructor calls.
**Example (C#)**:
```csharp
// Thickness - single value (uniform), two values (h, v), or four values (l, t, r, b)
new Button()
    .Margin(12)              // Thickness(12) - all sides
    .Padding(10, 20)         // Thickness(10, 20, 10, 20) - horizontal, vertical
    .CornerRadius(15)        // CornerRadius(15) - all corners

// Color / Brush - pass Color enum directly
new Button()
    .Foreground(Colors.Red)
    .Background(new SolidColorBrush().Color("#676767"))

// ImageSource - pass string path
new Image()
    .Source("ms-appx:///Assets/logo.png")
```
**Common Mistakes**:
- Writing `new Thickness(12)` when `.Margin(12)` suffices. The helper already converts the int.
- Using `.Margin(new Thickness(10, 20, 10, 20))` instead of the shorthand `.Margin(10, 20, 10, 20)`.
- Passing a hex string directly to `.Foreground("#FF0000")` without wrapping it in a Brush. Use `new SolidColorBrush().Color("#FF0000")` or a resource reference instead.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/GettingStarted.html

---

### Children and Content Helpers
**Rule**: Use `.Children(...)` for panel types (`StackPanel`, `Grid`) and `.Content(...)` for content controls (`Page`, `Button`, `Border`). Never mix them up.
**Why**: `.Children()` maps to `Panel.Children` (accepts multiple `UIElement` arguments). `.Content()` maps to `ContentControl.Content` (accepts a single object). Using the wrong one will not compile, but knowing the distinction avoids confusion when reading code.
**Example (C#)**:
```csharp
// Panel - use .Children() for multiple elements
new StackPanel()
    .Spacing(8)
    .Children(
        new TextBlock().Text("Title"),
        new TextBlock().Text("Subtitle"),
        new Button().Content("Action")
    )

// ContentControl - use .Content() for a single child
new Button()
    .Content("Click Me")

// Border - use .Child() for a single UIElement
new Border()
    .Background(Colors.LightGray)
    .CornerRadius(8)
    .Padding(16)
    .Child(
        new TextBlock().Text("Inside a border")
    )

// Page - use .Content() to set the page root
this.Content(
    new Grid()
        .RowDefinitions("Auto, *")
        .Children(
            new TextBlock().Text("Header").Grid(row: 0),
            new ListView().Grid(row: 1)
        )
);
```
**Common Mistakes**:
- Calling `.Content()` on a `StackPanel` or `Grid`. Panels do not have a `Content` property; use `.Children()`.
- Calling `.Children()` on a `Button` or `Border`. These are content controls; use `.Content()` or `.Child()`.
- Passing a single child to `.Children()` when a `ContentControl` wrapper would be more appropriate for semantic clarity.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/Overview.html

---

### Namespace Setup
**Rule**: Add the core C# Markup namespaces at the top of every page file. Include companion namespaces for Toolkit, Themes, or Navigation Markup packages when used.
**Why**: The fluent extension methods live in the namespace of the type they extend. Without the correct `using` directives, IntelliSense will not discover them and the code will not compile. A consistent set of imports across all pages eliminates friction.
**Example (C#)**:
```csharp
// Core C# Markup
using Uno.Extensions.Markup;

// WinUI controls and enums
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;
using Microsoft.UI.Xaml.Media;
using Microsoft.UI;

// Toolkit Markup (when UnoFeatures includes Toolkit)
using Uno.Toolkit.UI;                    // Toolkit controls
// Extensions are in the control's namespace, auto-discovered

// Themes / Material Markup
using Uno.Themes.Markup;                 // Theme.Brushes, Theme.Colors, Theme.Styles

// Navigation Markup (when UnoFeatures includes Navigation)
using Uno.Extensions.Navigation.UI;
```
**Common Mistakes**:
- Missing `using Uno.Extensions.Markup;`. This is required for all fluent extensions including `.Content()`, `.Children()`, `.Text()`, etc.
- Adding `using Microsoft.UI.Xaml.Markup;` (the XAML markup namespace) instead of `using Uno.Extensions.Markup;`. These are different namespaces with different purposes.
- Forgetting `using Uno.Themes.Markup;` when trying to use `Theme.Brushes.Primary.Default` or other strongly-typed theme accessors.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/GettingStarted.html

---

### Grid Row/Column Definitions Shorthand
**Rule**: Define Grid rows and columns using string shorthand passed to `.RowDefinitions()` and `.ColumnDefinitions()`. Use `"Auto"`, `"*"`, or numeric pixel values separated by commas.
**Why**: The string parser mirrors XAML's concise syntax and avoids verbose `new RowDefinition { Height = new GridLength(...) }` chains. This keeps Grid layouts readable at a glance.
**Example (C#)**:
```csharp
new Grid()
    .RowDefinitions("Auto, *, Auto")
    .ColumnDefinitions("200, *")
    .Children(
        new TextBlock()
            .Text("Header")
            .Grid(row: 0, column: 0, columnSpan: 2),
        new ListView()
            .Grid(row: 1, column: 0, columnSpan: 2),
        new Button()
            .Content("Footer Action")
            .Grid(row: 2, column: 1)
    )
```
**Common Mistakes**:
- Using semicolons or spaces without commas as separators. The parser expects comma-delimited values.
- Forgetting to assign `.Grid(row:, column:)` on child elements, causing them all to stack in row 0, column 0.
- Using `.Grid(0, 1)` without named parameters. Always use `row:` and `column:` for readability: `.Grid(row: 0, column: 1)`.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/AttachedProperties.html
