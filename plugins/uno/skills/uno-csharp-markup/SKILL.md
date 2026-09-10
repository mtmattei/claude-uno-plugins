---
name: uno-csharp-markup
description: "C# Markup for Uno Platform: code-first UI with fluent API, strongly-typed data binding, resources, styles, templates, and Uno Toolkit integration. Use when: (1) Building UI in C# instead of XAML, (2) Using fluent data binding with expression trees, (3) Applying styles and templates in C# Markup, (4) Integrating Uno Toolkit controls in C# Markup, (5) Setting up a C# Markup project from scratch, (6) Converting XAML patterns to C# Markup. Do NOT use for: XAML-based UI (see winui-xaml) or Toolkit control reference (see uno-toolkit)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation)"
metadata:
  version: "1.0.0"
---

# Uno Platform C# Markup

Declarative, fluent-style C# syntax for building Uno Platform UIs. Same underlying object model as XAML with compile-time validation, IntelliSense, and refactoring support.

## Setup

Select C# Markup in the project template:

```bash
dotnet new unoapp -markup csharp
```

Add `CSharpMarkup` to UnoFeatures for existing projects:

```xml
<UnoFeatures>
    CSharpMarkup;
    Material;
    Toolkit;
</UnoFeatures>
```

Pages use `.cs` files instead of `.xaml` + `.xaml.cs` pairs.

## Core Patterns

### Creating Controls and Setting Properties

Fluent API chains property setters. Type conversion is automatic for common types (`Thickness`, `Brush`, `Color`, `CornerRadius`, `FontFamily`).

```csharp
new TextBlock()
    .Margin(12)                              // auto-converts int to Thickness
    .TextAlignment(TextAlignment.Center)     // strongly typed enum
    .Text("Hello Uno Platform!")
```

### Children and Content

```csharp
new StackPanel()
    .VerticalAlignment(VerticalAlignment.Center)
    .HorizontalAlignment(HorizontalAlignment.Center)
    .Children(
        new TextBlock().Text("First"),
        new TextBlock().Text("Second"),
        new Button().Content("Click me")
    )
```

### DataContext and Binding

Use expression trees for strongly-typed binding. The `vm` parameter is a placeholder, not a live reference.

```csharp
public sealed partial class MainPage : Page
{
    public MainPage()
    {
        this.DataContext(new MainViewModel(), (page, vm) => page
            .Background(ThemeResource.Get<Brush>("ApplicationPageBackgroundThemeBrush"))
            .Content(
                new StackPanel()
                    .VerticalAlignment(VerticalAlignment.Center)
                    .Children(
                        new TextBlock()
                            .Text(() => vm.Count, count => $"Counter: {count}"),
                        new TextBox()
                            .Text(x => x.Bind(() => vm.Step).TwoWay()),
                        new Button()
                            .Command(() => vm.IncrementCommand)
                            .Content("Increment")
                    )
            )
        );
    }
}
```

**Binding modes**:
- Default (OneWay): `() => vm.Property`
- TwoWay: `.Text(x => x.Bind(() => vm.Property).TwoWay())`
- OneTime: `.Text(x => x.Bind(() => vm.Property).OneTime())`

**Without ViewModel instance** (type-only):
```csharp
this.DataContext<MainViewModel>((page, vm) => page
    .Content(...)
);
```

### Resources

Access theme and static resources with strongly-typed API:

```csharp
// Theme resource
this.Background(ThemeResource.Get<Brush>("ApplicationPageBackgroundThemeBrush"));

// Static resource
new TextBlock()
    .Style(StaticResource.Get<Style>("TitleLarge"));
```

### Styles

```csharp
new TextBlock()
    .Style(new Style<TextBlock>()
        .Setters(s => s
            .Foreground(ThemeResource.Get<Brush>("OnSurfaceBrush"))
            .FontSize(24)
        )
    )
```

### Templates

```csharp
new ListView()
    .ItemsSource(() => vm.Items)
    .ItemTemplate<MyItem>(item =>
        new StackPanel()
            .Orientation(Orientation.Horizontal)
            .Children(
                new TextBlock().Text(() => item.Name),
                new TextBlock().Text(() => item.Description)
            )
    )
```

## Uno Toolkit in C# Markup

Toolkit controls work with the same fluent API:

```csharp
using Uno.Toolkit.UI;

new AutoLayout()
    .Spacing(16)
    .Padding(24)
    .PrimaryAxisAlignment(AutoLayoutAlignment.Center)
    .Children(
        new NavigationBar().Content("Page Title"),
        new SafeArea()
            .Insets(SafeArea.InsetMask.Top | SafeArea.InsetMask.Bottom)
            .Child(
                new TextBlock().Text("Safe content")
            )
    )
```

### ResponsiveExtension in C# Markup

Responsive values require XAML-style markup extensions. In C# Markup, use adaptive logic or VisualStateManager instead:

```csharp
// Use VisualStateManager for responsive layouts in C# Markup
new Grid()
    .VisualStateManager(vsm => vsm
        .Group(g => g.Name("WidthStates")
            .State(s => s.Name("Narrow")
                .StateTriggers(new AdaptiveTrigger() { MinWindowWidth = 0 })
                .Setters(/* narrow layout */)
            )
            .State(s => s.Name("Wide")
                .StateTriggers(new AdaptiveTrigger() { MinWindowWidth = 800 })
                .Setters(/* wide layout */)
            )
        )
    )
```

## MVUX with C# Markup

C# Markup pairs naturally with MVUX models. Bind directly to Feed/State properties:

```csharp
this.DataContext<MainModel>((page, vm) => page
    .Content(
        new FeedView()
            .Source(() => vm.Items)
            .DataTemplate<ItemViewModel>(item =>
                new TextBlock().Text(() => item.Name)
            )
    )
);
```

## Common Mistakes

- Using string-based `{Binding}` syntax instead of expression-tree bindings
- Forgetting that `vm` is a placeholder, not a live object (don't call methods on it outside binding expressions)
- Setting `DataContext` in code-behind AND in the fluent chain (double initialization)
- Not adding `CSharpMarkup` to UnoFeatures (extension methods won't be generated)
- Mixing XAML and C# Markup in the same page (use one or the other per page)

## Related Skills

| Skill | Use instead when... |
|-------|-------------------|
| `winui-xaml` | Writing UI in XAML instead of C# Markup |
| `uno-toolkit` | Looking up Toolkit control APIs, properties, or XAML examples |
| `uno-platform-agent` | Setting up projects, MVVM/MVUX patterns, or navigation |
| `uno-extensions-services` | Configuring hosting, DI, authentication, or HTTP clients |

## XAML Equivalence and Limitations

For a full XAML-to-C# Markup mapping table, see [references/01-getting-started.md](references/01-getting-started.md). Key limitations: Hot Design supports XAML only (not C# Markup), `{utu:Responsive}` needs VisualStateManager alternatives, and `x:Uid` localization requires manual `ResourceLoader` usage.

## Detailed References

- [references/01-getting-started.md](references/01-getting-started.md) - Read when creating a C# Markup project or understanding the fluent API
- [references/02-binding-resources.md](references/02-binding-resources.md) - Read when implementing data binding, static/theme resources, or converters
- [references/03-styles-templates.md](references/03-styles-templates.md) - Read when defining styles, templates, or VisualStateManager in C#
- [references/04-toolkit-integration.md](references/04-toolkit-integration.md) - Read when using Uno Toolkit controls or responsive patterns in C# Markup
