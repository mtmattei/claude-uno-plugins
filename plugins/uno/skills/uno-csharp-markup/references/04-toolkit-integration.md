# C# Markup - Toolkit Integration and Limitations

Reference for Uno Toolkit controls in C# Markup, responsive patterns, MVUX FeedView binding, Material theme styles, and known limitations.

---

### Uno Toolkit Controls in C# Markup
**Rule**: Add `Uno.Toolkit.WinUI.Markup` (or `Uno.Toolkit.WinUI.Material.Markup` for Material) to access fluent extensions for Toolkit controls like `AutoLayout`, `NavigationBar`, `SafeArea`, `Card`, `Chip`, `TabBar`, and `DrawerControl`.
**Why**: Toolkit controls have their own DependencyProperties that require dedicated Markup extension packages. Without `Uno.Toolkit.WinUI.Markup`, you can instantiate Toolkit controls but cannot use fluent setters for their custom properties (e.g., `AutoLayout.Spacing`, `SafeArea.Insets`).
**Example (C#)**:
```csharp
// AutoLayout - flex-like container with spacing and padding
new AutoLayout()
    .PrimaryAxisAlignment(AutoLayoutAlignment.Center)
    .CounterAxisAlignment(AutoLayoutAlignment.Center)
    .Spacing(12)
    .Padding(16)
    .Children(
        new TextBlock()
            .Style(StaticResource.Get<Style>("TitleLarge"))
            .Text("Welcome"),
        new TextBlock()
            .Style(StaticResource.Get<Style>("BodyMedium"))
            .Text("Get started with Uno Platform"),
        new Button()
            .Content("Continue")
    )

// NavigationBar - app bar with title and actions
new NavigationBar()
    .Content("Settings")

// SafeArea - respects device notches and system bars
new SafeArea()
    .InsetMask(SafeArea.InsetMask.All)
    .Child(
        new AutoLayout()
            .PrimaryAxisAlignment(AutoLayoutAlignment.Start)
            .Spacing(8)
            .Padding(16)
            .Children(
                new TextBlock().Text("Content safe from notches")
            )
    )

// Card with explicit style (required per Uno Platform guidelines)
new Card()
    .Style(StaticResource.Get<Style>("ElevatedCardStyle"))
    .Content(
        new AutoLayout()
            .Spacing(8)
            .Padding(16)
            .Children(
                new TextBlock().Text("Card Title"),
                new TextBlock().Text("Card body content here")
            )
    )
```
**Common Mistakes**:
- Using `AutoLayout` without the Toolkit Markup package. The `Spacing` and `PrimaryAxisAlignment` fluent extensions will not be available.
- Forgetting to set an explicit `Style` on `Card`. Per Uno Platform guidelines, `Card` requires an explicit style (e.g., `ElevatedCardStyle`, `FilledCardStyle`, `OutlinedCardStyle`).
- Using `AppBarButton` outside a `CommandBar` or `NavigationBar`. Use a regular `Button` with a `FontIcon` or `ControlExtensions.Icon` instead.
**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/getting-started.html#using-c-markup

---

### Responsive Patterns Without Markup Extensions
**Rule**: Since `{Responsive}` markup extension is XAML-only, implement responsive layouts in C# Markup using `VisualStateManager` with `AdaptiveTrigger` for breakpoint-based changes, or use `SizeChanged` events for dynamic sizing.
**Why**: The `Responsive` markup extension (`{utu:Responsive}`) generates XAML-specific bindings that have no C# Markup equivalent. `AdaptiveTrigger` is the standard WinUI mechanism for responsive design and works identically in both XAML and C# Markup.
**Example (C#)**:
```csharp
public sealed partial class DashboardPage : Page
{
    public DashboardPage()
    {
        this.DataContext<DashboardViewModel>((page, vm) => page
            .Content(
                new Grid()
                    .ColumnDefinitions("*")
                    .RowDefinitions("Auto, *")
                    .VisualStateManager(vsm => vsm
                        .Group(g => g.Name("LayoutStates")
                            .State(s => s.Name("Compact")
                                .StateTriggers(new AdaptiveTrigger().MinWindowWidth(0))
                            )
                            .State(s => s.Name("Regular")
                                .StateTriggers(new AdaptiveTrigger().MinWindowWidth(720))
                            )
                            .State(s => s.Name("Wide")
                                .StateTriggers(new AdaptiveTrigger().MinWindowWidth(1280))
                            )
                        )
                    )
                    .Children(
                        new NavigationBar().Content("Dashboard").Grid(row: 0),
                        new AutoLayout()
                            .PrimaryAxisAlignment(AutoLayoutAlignment.Start)
                            .Spacing(16)
                            .Padding(16)
                            .Grid(row: 1)
                            .Children(
                                new TextBlock().Text("Content adapts to window size")
                            )
                    )
            )
        );
    }
}
```
**Common Mistakes**:
- Attempting to use `ResponsiveExtension` or `Responsive` in C# Markup code. This is a XAML-only markup extension and has no C# API.
- Handling all responsive logic in code-behind with `SizeChanged` events. Prefer declarative `AdaptiveTrigger` for standard breakpoints; use `SizeChanged` only for custom continuous calculations (e.g., scaling a value proportionally to width).
- Hardcoding pixel values for breakpoints without aligning to the standard breakpoint scale (0, 640, 1008, 1280).
**Uno Platform Notes**: For complex responsive scenarios, consider splitting into separate `UserControl` components for each layout variant and swapping them via `VisualStateManager`, rather than hiding/showing elements within a single crowded Grid.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/HowTo-CustomMarkupProject-VisualStates.html

---

### MVUX FeedView Binding
**Rule**: When using MVUX, bind `IFeed<T>` and `IListFeed<T>` to `FeedView` and list controls using the standard C# Markup binding syntax. MVUX models expose feeds as bindable properties automatically.
**Why**: MVUX `IFeed<T>` wraps async data with loading/error/data states. `FeedView` renders the appropriate template for each state. The C# Markup binding `() => vm.Feed` works because MVUX generates bindable proxy properties that the expression tree resolves into standard binding paths.
**Example (C#)**:
```csharp
// MVUX Model
public partial record DashboardModel
{
    public IFeed<UserProfile> Profile => Feed.Async(async ct =>
        await _userService.GetProfileAsync(ct));

    public IListFeed<Notification> Notifications => ListFeed.AsyncEnumerable(
        ct => _notificationService.GetNotificationsAsync(ct));
}

// Page with FeedView for single feed
this.DataContext<DashboardModel>((page, vm) => page
    .Content(
        new FeedView()
            .Source(() => vm.Profile)
            .ValueTemplate<UserProfile>(profile =>
                new AutoLayout()
                    .Spacing(8)
                    .Children(
                        new TextBlock()
                            .Style(StaticResource.Get<Style>("TitleLarge"))
                            .Text(() => profile.DisplayName),
                        new TextBlock()
                            .Text(() => profile.Email)
                    )
            )
    )
);

// ListView with IListFeed
new ListView()
    .ItemsSource(() => vm.Notifications)
    .ItemTemplate<Notification>(n =>
        new AutoLayout()
            .Orientation(Orientation.Horizontal)
            .Spacing(12)
            .Padding(12)
            .Children(
                new FontIcon().Glyph("\uE7E7"),
                new TextBlock().Text(() => n.Message)
            )
    )
```
**Common Mistakes**:
- Binding an `IFeed<T>` directly to a `TextBlock.Text`. Use `FeedView` to handle loading/error states, or extract the value through a `State<T>` if you only need the resolved value.
- Forgetting that MVUX models use `partial record` (not `class`). The source generator requires the `partial record` declaration to generate bindable proxies.
- Using `.TwoWay()` on a Feed binding. Feeds are read-only data streams. Use `IState<T>` for read-write state.
**Reference**: https://platform.uno/docs/articles/getting-started/counterapp/get-started-counter-csharp-mvux.html

---

### Uno.Themes.WinUI.Markup for Material Styles
**Rule**: Add the `Uno.Themes.WinUI.Markup` package to access `Theme.Brushes`, `Theme.Colors`, and `Theme.Styles` for strongly-typed Material Design resource access. Prefer these over raw `ThemeResource.Get<T>("key")` strings.
**Why**: `Theme.Brushes.Primary.Default` is discoverable via IntelliSense and verified at compile time. `ThemeResource.Get<Brush>("PrimaryBrush")` requires memorizing key names and fails silently at runtime if misspelled. The strongly typed API reduces errors and speeds up development.
**Example (C#)**:
```csharp
// Strongly typed Material brushes
new TextBlock()
    .Foreground(Theme.Brushes.OnSurface.Default)

new Border()
    .Background(Theme.Brushes.Surface.Default)
    .BorderBrush(Theme.Brushes.Outline.Default)
    .BorderThickness(1)

// Strongly typed Material styles
new Button()
    .Style(Theme.Styles.Button.Filled)
    .Content("Primary Action")

new Button()
    .Style(Theme.Styles.Button.Outlined)
    .Content("Secondary Action")

// Lightweight styling - override specific resources on a control
new Button()
    .Style(Theme.Styles.Button.Filled)
    .Resources(r => r
        .Add("FilledButtonForeground", Theme.Brushes.OnPrimary.Default)
        .Add("FilledButtonBackground", Theme.Brushes.Primary.Default)
    )
    .Content("Custom Filled")

// Using Theme resources in Style Setters
new Style<Button>()
    .Setters(s => s.Background(Theme.Brushes.Primary.Default))
```
**Common Mistakes**:
- Using `Theme.Brushes` without adding the `Uno.Themes.WinUI.Markup` NuGet package. The `Theme` static class is not available from the base Markup package alone.
- Mixing `Theme.Brushes` calls with hardcoded hex color strings. This breaks theme consistency and dark mode support.
- Using `Theme.Styles` for a control type that does not have a Material style variant. Not all WinUI controls have Material-themed styles. Check the Uno Themes documentation for supported controls.
**Uno Platform Notes**: Cupertino themes do not currently have C# Markup support. Material has full C# Markup support via `Uno.Themes.WinUI.Markup` (providing `Theme.Brushes`, `Theme.Styles`). Fluent has a companion `.Markup` package but verify feature parity with Material before relying on strongly-typed helpers.
**Reference**: https://platform.uno/docs/articles/external/uno.themes/doc/lightweight-styling.html#c-markup

---

### Limitation: Hot Design
**Rule**: Do not expect Hot Design (visual designer) support for C# Markup pages. Hot Design currently supports XAML pages only.
**Why**: Hot Design parses and renders XAML markup in real time. C# Markup pages are compiled C# code that requires execution to produce a visual tree, which Hot Design cannot interpret at design time.
**Example (C#)**:
```csharp
// This page WILL NOT appear in Hot Design
public sealed partial class ProfilePage : Page
{
    public ProfilePage()
    {
        this.Content(
            new AutoLayout().Children(
                new TextBlock().Text("Profile Page")
            )
        );
    }
}

// WORKAROUND: If Hot Design is critical for a specific page,
// consider defining that page in XAML and using C# Markup for others.
// XAML and C# Markup pages can coexist in the same project.
```
**Common Mistakes**:
- Expecting a live preview of C# Markup pages in the IDE designer pane. No visual designer currently supports C# Markup rendering.
- Converting all pages to C# Markup in a project that heavily relies on Hot Design for designer-developer collaboration. Keep high-churn design pages in XAML if Hot Design workflow is essential.
**Reference**: https://platform.uno/docs/articles/features/using-markup.html

---

### Limitation: x:Uid and Localization
**Rule**: C# Markup does not support `x:Uid` automatic resource binding. Use `ResourceLoader` manually to retrieve localized strings and bind or set them explicitly.
**Why**: `x:Uid` is an XAML-specific directive that the XAML parser resolves at load time by matching resource keys to property names (e.g., `MyButton.Content`). Since C# Markup bypasses the XAML parser entirely, this automatic resolution does not occur.
**Example (C#)**:
```csharp
// Manual localization in C# Markup
using Windows.ApplicationModel.Resources;

public sealed partial class SettingsPage : Page
{
    private static readonly ResourceLoader _resources =
        ResourceLoader.GetForViewIndependentUse();

    public SettingsPage()
    {
        this.Content(
            new AutoLayout()
                .Spacing(12)
                .Padding(16)
                .Children(
                    new TextBlock()
                        .Text(_resources.GetString("SettingsPage_Title")),
                    new Button()
                        .Content(_resources.GetString("SettingsPage_SaveButton")),
                    new TextBlock()
                        .Text(_resources.GetString("SettingsPage_Description"))
                )
        );
    }
}
```
In `Strings/en/Resources.resw`:
```
SettingsPage_Title         = Settings
SettingsPage_SaveButton    = Save Changes
SettingsPage_Description   = Configure your application preferences.
```
**Common Mistakes**:
- Assuming `x:Uid` works in C# Markup. It does not. Every localized string must be loaded explicitly.
- Calling `ResourceLoader.GetForViewIndependentUse()` on every property set. Create a single static instance per page or a shared helper class.
- Forgetting to create `.resw` files for each supported locale. `ResourceLoader.GetString()` returns an empty string if the key is missing, with no compile-time warning.
**Uno Platform Notes**: Consider creating a helper extension method (e.g., `.Localized("Key")`) that wraps `ResourceLoader` to reduce boilerplate across pages.
**Reference**: https://platform.uno/docs/articles/features/using-markup.html

---

### Limitation: ResponsiveExtension in C# Markup
**Rule**: The `{utu:Responsive}` XAML markup extension is not available in C# Markup. Use `VisualStateManager` with `AdaptiveTrigger` or implement responsive logic through a ViewModel property.
**Why**: `{utu:Responsive Narrow=X, Wide=Y}` is a XAML markup extension that returns different values based on window width at parse time. Markup extensions are a XAML parser concept with no C# Markup counterpart. The equivalent C# approach is declarative visual states or reactive ViewModel properties.
**Example (C#)**:
```csharp
// Option 1: AdaptiveTrigger (declarative, preferred)
// See the VisualStateManager rule in 03-styles-templates.md

// Option 2: ViewModel-driven responsive property (when logic is complex)
// In ViewModel:
public partial class ShellViewModel : ObservableObject
{
    [ObservableProperty]
    private Orientation _listOrientation = Orientation.Vertical;

    public void OnWindowSizeChanged(double width)
    {
        ListOrientation = width >= 720
            ? Orientation.Horizontal
            : Orientation.Vertical;
    }
}

// In Page:
new StackPanel()
    .Orientation(x => x.Bind(() => vm.ListOrientation))
```
**Common Mistakes**:
- Trying to instantiate `ResponsiveExtension` as a C# class and pass it to a property setter. It is a XAML-only construct.
- Over-engineering responsive behavior with complex `SizeChanged` handlers when a simple two-state `AdaptiveTrigger` would suffice.
- Forgetting that `AdaptiveTrigger.MinWindowWidth` uses effective pixels, not physical pixels. On high-DPI displays, the effective width is smaller than the physical resolution.
**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Markup/HowTo-CustomMarkupProject-VisualStates.html
