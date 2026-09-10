# Interaction Extensions

## Impact: HIGH
CommandExtensions, InputExtensions, SelectorExtensions, FlipViewExtensions, and ItemsRepeaterExtensions eliminate code-behind for common UI interactions. Using the wrong event assumption, forgetting prerequisite properties, or mismatching template root types causes commands to silently never fire.

---

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

## CommandExtensions

**Impact: CRITICAL**

Attach `utu:CommandExtensions.Command` to any UIElement to bind a command to its natural interaction. The extension auto-selects the correct event and passes the appropriate default parameter per control type.

### Default Event and Parameter Per Control

| Control | Event | Default Parameter |
|---------|-------|-------------------|
| `TextBox` | Enter key | `Text` (string) |
| `PasswordBox` | Enter key | `Password` (string) |
| `ToggleSwitch` | Toggled | `IsOn` (bool) |
| `ListView` | ItemClick | Clicked item |
| `NavigationView` | ItemInvoked | Invoked item |
| `ItemsRepeater` | Tapped on item | Item DataContext |
| `UIElement` (any) | Tapped | `null` (or CommandParameter if set) |

### CORRECT
```xml
<!-- Tappable card (UIElement.Tapped, parameter is null) -->
<Border Background="{ThemeResource SurfaceBrush}"
        CornerRadius="12" Padding="16"
        utu:CommandExtensions.Command="{Binding OpenDetailCommand}">
    <TextBlock Text="Tap to open" Style="{StaticResource BodyLarge}" />
</Border>

<!-- ListView item click (parameter is the clicked item) -->
<ListView ItemsSource="{Binding People}"
          IsItemClickEnabled="True"
          utu:CommandExtensions.Command="{Binding SelectPersonCommand}" />

<!-- TextBox on Enter key (parameter is Text) -->
<TextBox PlaceholderText="Search..."
         utu:CommandExtensions.Command="{Binding SearchCommand}" />

<!-- PasswordBox on Enter (parameter is Password, also dismisses keyboard) -->
<PasswordBox PlaceholderText="Password"
             utu:CommandExtensions.Command="{Binding LoginCommand}" />

<!-- ToggleSwitch toggled (parameter is IsOn bool) -->
<ToggleSwitch Header="Dark Mode"
              utu:CommandExtensions.Command="{Binding ToggleDarkModeCommand}" />

<!-- NavigationView item invoked -->
<NavigationView utu:CommandExtensions.Command="{Binding NavigateCommand}">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" />
        <NavigationViewItem Content="Settings" />
    </NavigationView.MenuItems>
</NavigationView>

<!-- ItemsRepeater item tap (parameter is item DataContext) -->
<muxc:ItemsRepeater ItemsSource="{Binding Items}"
                    utu:CommandExtensions.Command="{Binding ItemTappedCommand}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" Padding="12" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>
```

### WRONG
```xml
<!-- WRONG: ListView without IsItemClickEnabled — command will never fire -->
<ListView ItemsSource="{Binding People}"
          utu:CommandExtensions.Command="{Binding SelectPersonCommand}" />
<!-- FIX: Add IsItemClickEnabled="True" -->

<!-- WRONG: Setting CommandParameter when you want the automatic parameter -->
<!-- CommandParameter overrides the auto parameter (SelectedItem, Text, IsOn, etc.) -->
<ListView ItemsSource="{Binding People}"
          IsItemClickEnabled="True"
          utu:CommandExtensions.Command="{Binding SelectPersonCommand}"
          utu:CommandExtensions.CommandParameter="fixed-value" />
<!-- FIX: Omit CommandParameter to get the clicked item automatically -->

<!-- WRONG: Using code-behind event handler when CommandExtensions supports the interaction -->
<ToggleSwitch Header="Dark Mode" Toggled="OnDarkModeToggled" />
<!-- FIX: Use utu:CommandExtensions.Command="{Binding ToggleDarkModeCommand}" -->
```

---

## InputExtensions

**Impact: HIGH**

Attached properties for form input UX: auto-advance focus, dismiss keyboard, and customize the soft keyboard return button.

### Properties

| Property | Type | Effect |
|----------|------|--------|
| `AutoFocusNext` | bool | Move focus to next focusable element on Enter |
| `AutoFocusNextElement` | Control | Move focus to specific element on Enter |
| `AutoDismiss` | bool | Dismiss keyboard on Enter |
| `ReturnType` | enum | Customize soft keyboard return key |

### ReturnType Enum Values

| Value | Keyboard Button Label |
|-------|----------------------|
| `Default` | Platform default |
| `Done` | "Done" |
| `Go` | "Go" |
| `Next` | "Next" |
| `Search` | "Search" |
| `Send` | "Send" |

### CORRECT
```xml
<utu:AutoLayout Orientation="Vertical" Spacing="12" Padding="16">
    <!-- First field: auto-advance to next on Enter -->
    <TextBox PlaceholderText="First Name"
             utu:InputExtensions.AutoFocusNext="True"
             utu:InputExtensions.ReturnType="Next" />

    <!-- Second field: advance to a specific element (skip order) -->
    <TextBox x:Name="LastNameInput"
             PlaceholderText="Last Name"
             utu:InputExtensions.AutoFocusNextElement="{Binding ElementName=EmailInput}"
             utu:InputExtensions.ReturnType="Next" />

    <TextBox x:Name="EmailInput"
             PlaceholderText="Email"
             utu:InputExtensions.AutoFocusNext="True"
             utu:InputExtensions.ReturnType="Next" />

    <!-- Last field: dismiss keyboard and fire command -->
    <PasswordBox PlaceholderText="Password"
                 utu:InputExtensions.AutoDismiss="True"
                 utu:InputExtensions.ReturnType="Done"
                 utu:CommandExtensions.Command="{Binding SubmitCommand}" />
</utu:AutoLayout>
```

### WRONG
```xml
<!-- WRONG: Both AutoFocusNext and AutoFocusNextElement on the same control -->
<!-- AutoFocusNextElement takes precedence, making AutoFocusNext pointless -->
<TextBox utu:InputExtensions.AutoFocusNext="True"
         utu:InputExtensions.AutoFocusNextElement="{Binding ElementName=NextField}" />
<!-- FIX: Use only one — AutoFocusNextElement if you need specific targeting -->

<!-- WRONG: Expecting AutoFocusNext to follow visual layout order -->
<!-- AutoFocusNext uses FocusManager.FindNextFocusableElement (visual tree order) -->
<!-- If your form layout is non-sequential, use AutoFocusNextElement instead -->
```

---

## SelectorExtensions

**Impact: MEDIUM**

Adds `SelectionMode` enhancements and a `SelectedCommand` to Selector-based controls (ComboBox, ListView, etc.).

### SingleOrNone Mode

The `SingleOrNone` selection mode allows tapping a selected item to deselect it (unlike the built-in `Single` mode which always requires a selection).

### CORRECT
```xml
<!-- Allow deselecting the current selection by tapping it again -->
<ListView ItemsSource="{Binding Options}"
          utu:SelectorExtensions.SelectionMode="SingleOrNone"
          SelectedItem="{Binding SelectedOption, Mode=TwoWay}" />

<!-- Fire a command when selection changes -->
<ListView ItemsSource="{Binding Items}"
          utu:SelectorExtensions.SelectedCommand="{Binding ItemSelectedCommand}" />
```

### WRONG
```xml
<!-- WRONG: Using SelectionMode="Single" when you need nullable selection -->
<ListView ItemsSource="{Binding Options}"
          SelectionMode="Single"
          SelectedItem="{Binding SelectedOption, Mode=TwoWay}" />
<!-- User cannot deselect. FIX: Use utu:SelectorExtensions.SelectionMode="SingleOrNone" -->
```

---

## FlipViewExtensions

**Impact: LOW**

Adds previous/next navigation arrow buttons to `FlipView`. Arrows auto-show on desktop for mouse interaction.

### CORRECT
```xml
<FlipView ItemsSource="{Binding Images}"
          utu:FlipViewExtensions.PreviousButtonVisibility="Visible"
          utu:FlipViewExtensions.NextButtonVisibility="Visible">
    <FlipView.ItemTemplate>
        <DataTemplate>
            <Image Source="{Binding Url}" Stretch="UniformToFill" />
        </DataTemplate>
    </FlipView.ItemTemplate>
</FlipView>
```

---

## ItemsRepeaterExtensions

**Impact: HIGH**

Adds selection support and incremental loading to `ItemsRepeater`, which has neither built-in.

### Selection

| Property | Type | Modes |
|----------|------|-------|
| `SelectionMode` | enum | `None`, `SingleOrNone`, `Single`, `Multiple` |
| `SelectedItem` | object | Single selection binding |
| `SelectedItems` | object[] | Multiple selection binding (must be `object[]`) |
| `SelectedIndex` | int | Single selection by index |
| `SelectedIndexes` | int[] | Multiple selection by index |

### Template Root Constraint

**Impact: CRITICAL** -- The `ItemTemplate` root element **must** be one of: `SelectorItem`, `ToggleButton`, `Chip`, `ListViewItem`, `CheckBox`, or `RadioButton`. Other controls will not show selection visuals and selection will silently fail.

### CORRECT
```xml
<!-- Single selection with ListViewItem template root -->
<muxc:ItemsRepeater ItemsSource="{Binding Options}"
                    utu:ItemsRepeaterExtensions.SelectionMode="Single"
                    utu:ItemsRepeaterExtensions.SelectedItem="{Binding SelectedOption, Mode=TwoWay}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <ListViewItem Content="{Binding Name}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>

<!-- Multiple selection with CheckBox -->
<muxc:ItemsRepeater ItemsSource="{Binding Filters}"
                    utu:ItemsRepeaterExtensions.SelectionMode="Multiple"
                    utu:ItemsRepeaterExtensions.SelectedItems="{Binding SelectedFilters, Mode=TwoWay}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <CheckBox Content="{Binding Label}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>

<!-- Chip-based filter selection -->
<muxc:ItemsRepeater ItemsSource="{Binding Tags}"
                    utu:ItemsRepeaterExtensions.SelectionMode="Multiple"
                    utu:ItemsRepeaterExtensions.SelectedItems="{Binding SelectedTags, Mode=TwoWay}">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <utu:Chip Content="{Binding Name}"
                      Style="{StaticResource FilterChipStyle}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>
```

```csharp
// ViewModel: SelectedItems must be object[], not List<T> or ObservableCollection<T>
public object[] SelectedFilters { get; set; }
public object[] SelectedTags { get; set; }

// SelectedItem can use the concrete type
public OptionModel SelectedOption { get; set; }
```

### WRONG
```xml
<!-- WRONG: Template root is TextBlock — not a selectable control -->
<muxc:ItemsRepeater utu:ItemsRepeaterExtensions.SelectionMode="Single">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>
<!-- FIX: Wrap in ListViewItem, or use Chip, CheckBox, ToggleButton, RadioButton -->

<!-- WRONG: RadioButton with Multiple selection — RadioButton only supports single -->
<muxc:ItemsRepeater utu:ItemsRepeaterExtensions.SelectionMode="Multiple">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <RadioButton Content="{Binding Name}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>
<!-- FIX: Use CheckBox or Chip for multiple selection -->

<!-- WRONG: Manually binding IsChecked/IsSelected on template root -->
<!-- The extension manages these automatically -->
<DataTemplate>
    <CheckBox Content="{Binding Label}" IsChecked="{Binding IsSelected}" />
</DataTemplate>
<!-- FIX: Remove IsChecked binding — let the extension handle it -->
```

```csharp
// WRONG: SelectedItems bound to List<T> — must be object[]
public List<FilterModel> SelectedFilters { get; set; }

// WRONG: SelectedItems bound to ObservableCollection<T> — must be object[]
public ObservableCollection<FilterModel> SelectedFilters { get; set; }
```

### Incremental Loading

**Impact: MEDIUM**

| Property | Type | Purpose |
|----------|------|---------|
| `SupportsIncrementalLoading` | bool | Enable incremental loading |
| `DataFetchSize` | double | Pages of data to prefetch (multiplier of viewport) |
| `IncrementalLoadingThreshold` | double | Viewport distance threshold to trigger fetch |

### CORRECT
```xml
<muxc:ItemsRepeater ItemsSource="{Binding Items}"
                    utu:ItemsRepeaterExtensions.SupportsIncrementalLoading="True"
                    utu:ItemsRepeaterExtensions.DataFetchSize="2"
                    utu:ItemsRepeaterExtensions.IncrementalLoadingThreshold="1">
    <muxc:ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <ListViewItem Content="{Binding Title}" />
        </DataTemplate>
    </muxc:ItemsRepeater.ItemTemplate>
</muxc:ItemsRepeater>
```

The `ItemsSource` must implement `ISupportIncrementalLoading` for incremental loading to function.

---

## References

- [CommandExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/command-extensions.html)
- [InputExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/Input-extensions.html)
- [SelectorExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/Selector-extensions.html)
- [FlipViewExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/FlipView-extensions.html)
- [ItemsRepeaterExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/itemsrepeater-extensions.html)
