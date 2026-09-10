# Navigation Controls

## Impact: HIGH
NavigationBar, TabBar, SegmentedControl, ExtendedSplashScreen, and LoadingView provide cross-platform navigation chrome and loading states. Misusing native vs XAML mode, applying styles to items instead of containers, or placing Region.Attached inside splash screens causes silent rendering failures.

---

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

## NavigationBar

**Impact: HIGH**

NavigationBar maps to native UINavigationBar (iOS) and Toolbar (Android). It has three slots: `MainCommand` (leading button), `Content` (title), and `PrimaryCommands`/`SecondaryCommands` (trailing actions). Only `AppBarButton` is supported in command slots.

### CORRECT
```xml
<utu:NavigationBar Content="Page Title">
    <utu:NavigationBar.MainCommand>
        <AppBarButton Label="Back">
            <AppBarButton.Icon>
                <BitmapIcon UriSource="ms-appx:///Assets/Icons/arrow_back.png" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.MainCommand>
    <utu:NavigationBar.PrimaryCommands>
        <AppBarButton Label="Search">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE721;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.PrimaryCommands>
    <utu:NavigationBar.SecondaryCommands>
        <AppBarButton Label="Settings">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE713;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.SecondaryCommands>
</utu:NavigationBar>
```

### WRONG
```xml
<!-- WRONG: AppBarToggleButton is not supported in commands -->
<utu:NavigationBar.PrimaryCommands>
    <AppBarToggleButton Label="Toggle" />
</utu:NavigationBar.PrimaryCommands>

<!-- WRONG: Setting Height on mobile — height is fixed by platform (44pt iOS, 48dp Android) -->
<utu:NavigationBar Content="Title" Height="80" />

<!-- WRONG: Using AppBarButton standalone outside NavigationBar/CommandBar -->
<AppBarButton Label="Action" />
<!-- FIX: Use regular Button with an icon for standalone actions -->
```

---

## NavigationBar Windows vs Native Mode

**Impact: HIGH**

On iOS/Android the default rendering is **Native mode** (platform navigation bar). On Windows/WASM, it renders as a **XAML control**. Use `XamlDefaultNavigationBar` style to force XAML rendering on all platforms for full template customization.

### CORRECT
```xml
<!-- Native mode (default on mobile): platform-native transitions and styling -->
<utu:NavigationBar Content="Detail" />

<!-- XAML mode (full template control on desktop) -->
<utu:NavigationBar Content="Detail"
                   Style="{StaticResource XamlDefaultNavigationBar}" />
```

### Platform Notes
- **iOS**: SecondaryCommands are not supported. Width should always be `NaN` with `HorizontalAlignment="Stretch"`.
- **Android**: `HorizontalContentAlignment` supports only `Stretch` and `Left`.
- **Windows XAML mode**: Set `IsDynamicOverflowEnabled="False"` for proper HorizontalContentAlignment.

---

## NavigationBar MainCommandMode

**Impact: HIGH**

`MainCommandMode` controls the leading button behavior: `Back` (default) auto-shows a back arrow when the Frame has back stack entries. `Action` disables automatic back navigation for custom behavior (hamburger menu, close, confirm).

### CORRECT
```xml
<!-- Default: automatic back navigation when Frame has back stack -->
<utu:NavigationBar Content="Detail Page" />

<!-- Hamburger menu: Action mode with custom command -->
<utu:NavigationBar Content="Home"
                   MainCommandMode="Action">
    <utu:NavigationBar.MainCommand>
        <AppBarButton Command="{Binding ToggleMenuCommand}">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE700;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.MainCommand>
</utu:NavigationBar>

<!-- Custom back icon on non-mobile platforms -->
<Application.Resources>
    <x:String x:Key="NavigationBarBackIconData">M20,11V13H8L13.5,18.5L12.08,19.92L4.16,12L12.08,4.08L13.5,5.5L8,11H20Z</x:String>
</Application.Resources>
```

### WRONG
```xml
<!-- WRONG: MainCommandMode is Back (default) but using MainCommand for a hamburger menu -->
<!-- This causes unintended back navigation on tap -->
<utu:NavigationBar Content="Home">
    <utu:NavigationBar.MainCommand>
        <AppBarButton Command="{Binding ToggleMenuCommand}">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE700;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </utu:NavigationBar.MainCommand>
</utu:NavigationBar>
<!-- FIX: Add MainCommandMode="Action" -->
```

---

## TabBar and TabBarItem

**Impact: HIGH**

TabBar implements Material lateral navigation with built-in selection management. Style must be applied to the **TabBar container**, not individual TabBarItems. Each TabBarItem supports `Icon`, `Content` (label), and `Badge`.

### Available Styles (apply to TabBar, not TabBarItem)

| Style | Use Case |
|-------|----------|
| `BottomTabBarStyle` | Bottom navigation bar (includes SafeArea) |
| `TopTabBarStyle` | Top tab strip |
| `ColoredTopTabBarStyle` | Top tabs with colored background |
| `VerticalTabBarStyle` | Vertical side navigation |

### CORRECT
```xml
<!-- Bottom navigation: style on the container, icons on items -->
<utu:TabBar Style="{StaticResource BottomTabBarStyle}"
            SelectedIndex="0">
    <utu:TabBar.Items>
        <utu:TabBarItem Content="Home">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE80F;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
        <utu:TabBarItem Content="Search">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE721;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
        <utu:TabBarItem Content="Profile"
                        utu:TabBarItemExtensions.BadgeValue="3">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE77B;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
    </utu:TabBar.Items>
</utu:TabBar>

<!-- Top tabs -->
<utu:TabBar Style="{StaticResource TopTabBarStyle}">
    <utu:TabBar.Items>
        <utu:TabBarItem Content="All" />
        <utu:TabBarItem Content="Active" />
        <utu:TabBarItem Content="Completed" />
    </utu:TabBar.Items>
</utu:TabBar>

<!-- Vertical side tabs -->
<utu:TabBar Style="{StaticResource VerticalTabBarStyle}">
    <utu:TabBar.Items>
        <utu:TabBarItem Content="Dashboard">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE80F;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
        <utu:TabBarItem Content="Reports">
            <utu:TabBarItem.Icon>
                <FontIcon Glyph="&#xE9F9;" />
            </utu:TabBarItem.Icon>
        </utu:TabBarItem>
    </utu:TabBar.Items>
</utu:TabBar>
```

### WRONG
```xml
<!-- WRONG: Style applied to individual items instead of container -->
<utu:TabBar>
    <utu:TabBar.Items>
        <utu:TabBarItem Content="Home" Style="{StaticResource BottomTabBarStyle}" />
    </utu:TabBar.Items>
</utu:TabBar>

<!-- WRONG: Bottom nav without icons — Material Design requires icons -->
<utu:TabBar Style="{StaticResource BottomTabBarStyle}">
    <utu:TabBar.Items>
        <utu:TabBarItem Content="Home" />
    </utu:TabBar.Items>
</utu:TabBar>

<!-- WRONG: More than 5 items in bottom navigation (Material Design guideline) -->
```

---

## TabBar SelectionIndicator

**Impact: MEDIUM**

TabBar supports a selection indicator (underline or pill) via the `SelectionIndicatorContent`, `SelectionIndicatorTransitionMode`, and `SelectionIndicatorPresenterStyle` properties, or via the built-in styles that include indicators by default.

### CORRECT
```xml
<!-- TopTabBarStyle includes an underline indicator by default -->
<utu:TabBar Style="{StaticResource TopTabBarStyle}">
    <utu:TabBar.Items>
        <utu:TabBarItem Content="Tab 1" />
        <utu:TabBarItem Content="Tab 2" />
    </utu:TabBar.Items>
</utu:TabBar>

<!-- Programmatic selection -->
<utu:TabBar x:Name="MyTabBar"
            Style="{StaticResource BottomTabBarStyle}"
            SelectedIndex="1">
    <!-- Items... -->
</utu:TabBar>
```

---

## SegmentedControl (TabBar with Segmented Styles)

**Impact: MEDIUM**

A SegmentedControl is a TabBar styled with segmented appearance for mutually exclusive option selection (e.g., filter toggles). Use `MaterialSegmentedStyle` or `MaterialSegmentedItemStyle`.

### CORRECT
```xml
<!-- Segmented control for exclusive options -->
<utu:TabBar Style="{StaticResource MaterialSegmentedStyle}">
    <utu:TabBar.Items>
        <utu:TabBarItem Content="Day" />
        <utu:TabBarItem Content="Week" />
        <utu:TabBarItem Content="Month" />
    </utu:TabBar.Items>
</utu:TabBar>
```

### WRONG
```xml
<!-- WRONG: Using RadioButton group when TabBar segmented style exists -->
<StackPanel Orientation="Horizontal">
    <RadioButton Content="Day" GroupName="Period" />
    <RadioButton Content="Week" GroupName="Period" />
    <RadioButton Content="Month" GroupName="Period" />
</StackPanel>
<!-- FIX: Use TabBar with MaterialSegmentedStyle for Material Design compliance -->
```

---

## ExtendedSplashScreen

**Impact: HIGH**

ExtendedSplashScreen extends the native splash screen with a loading indicator while async initialization runs. It is a derivative of `LoadingView`. On Android, you **must** call `Init()` from `MainActivity.OnCreate` before `base.OnCreate`.

### CORRECT
```xml
<!-- Shell.xaml: ExtendedSplashScreen wrapping the main content -->
<utu:ExtendedSplashScreen x:Name="Splash"
                          HorizontalAlignment="Stretch"
                          VerticalAlignment="Stretch"
                          HorizontalContentAlignment="Stretch"
                          VerticalContentAlignment="Stretch">
    <utu:ExtendedSplashScreen.LoadingContentTemplate>
        <DataTemplate>
            <ProgressRing IsActive="True"
                          Width="40" Height="40" />
        </DataTemplate>
    </utu:ExtendedSplashScreen.LoadingContentTemplate>

    <!-- Main app content (shown after loading completes) -->
    <Grid>
        <!-- App content here -->
    </Grid>
</utu:ExtendedSplashScreen>
```

```csharp
// Android: MainActivity.cs — MUST call Init before base.OnCreate
protected override void OnCreate(Bundle savedInstanceState)
{
    ExtendedSplashScreen.Init(this);  // Must be BEFORE base.OnCreate
    base.OnCreate(savedInstanceState);
}
```

### WRONG
```csharp
// WRONG: Init called after base.OnCreate — splash will not display correctly
protected override void OnCreate(Bundle savedInstanceState)
{
    base.OnCreate(savedInstanceState);
    ExtendedSplashScreen.Init(this);  // Too late
}
```

```xml
<!-- WRONG: Using Region.Attached="True" inside ExtendedSplashScreen -->
<!-- Navigation regions cannot be placed inside the splash screen -->
<utu:ExtendedSplashScreen>
    <Grid uen:Region.Attached="True">
        <!-- This will fail: navigation is not ready during splash -->
    </Grid>
</utu:ExtendedSplashScreen>
<!-- FIX: Place Region.Attached on the content that loads AFTER splash completes -->
```

---

## LoadingView

**Impact: MEDIUM**

LoadingView displays a loading indicator while an async operation executes. The `Source` property binds to any `ILoadable` (such as `AsyncCommand`). Use `CompositeLoadableSource` to combine multiple loadable sources.

### CORRECT
```xml
<!-- LoadingView bound to an ILoadable command -->
<utu:LoadingView Source="{Binding LoadDataCommand}">
    <utu:LoadingView.LoadingContent>
        <StackPanel HorizontalAlignment="Center"
                    VerticalAlignment="Center"
                    Spacing="12">
            <ProgressRing IsActive="True" Width="32" Height="32" />
            <TextBlock Text="Loading data..."
                       Style="{StaticResource BodyMedium}" />
        </StackPanel>
    </utu:LoadingView.LoadingContent>

    <!-- Content shown when loading completes -->
    <ListView ItemsSource="{Binding Items}" />
</utu:LoadingView>

<!-- CompositeLoadableSource: multiple async operations -->
<utu:LoadingView>
    <utu:LoadingView.Source>
        <utu:CompositeLoadableSource>
            <utu:LoadableSource Source="{Binding LoadUsersCommand}" />
            <utu:LoadableSource Source="{Binding LoadSettingsCommand}" />
        </utu:CompositeLoadableSource>
    </utu:LoadingView.Source>

    <utu:LoadingView.LoadingContent>
        <ProgressRing IsActive="True" />
    </utu:LoadingView.LoadingContent>

    <!-- Shown only when ALL sources finish loading -->
    <Grid>
        <!-- Content here -->
    </Grid>
</utu:LoadingView>
```

### WRONG
```xml
<!-- WRONG: Binding Source to a non-ILoadable (e.g., a bool property) -->
<utu:LoadingView Source="{Binding IsLoading}">
    <!-- Source must implement ILoadable, not be a bool -->
</utu:LoadingView>
<!-- FIX: Use an AsyncCommand or wrap in a class implementing ILoadable -->
```

---

## References

- [NavigationBar](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/NavigationBar.html)
- [TabBar and TabBarItem](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/TabBarAndTabBarItem.html)
- [ExtendedSplashScreen](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ExtendedSplashScreen.html)
- [LoadingView](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/LoadingView.html)
