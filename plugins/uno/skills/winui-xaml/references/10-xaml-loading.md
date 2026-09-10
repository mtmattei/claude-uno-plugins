# XAML Loading Skills - WinUI 3 XAML

## XAML-001: Use x:Load for Conditional UI

**Rule**: Use `x:Load="False"` instead of `Visibility="Collapsed"` for UI elements that may not be shown. Bind x:Load to a boolean for dynamic control.

**Why**: `Visibility="Collapsed"` still creates the element and its children in memory. `x:Load="False"` prevents creation entirely, saving memory and startup time. Each deferred element saves ~600 bytes plus the cost of its children.

**Example (Correct)**:
```xml
<!-- Deferred loading with binding -->
<Grid x:Name="AdvancedSettings"
      x:Load="{x:Bind ViewModel.ShowAdvancedSettings, Mode=OneWay}">
    <!-- Complex settings UI not created until ShowAdvancedSettings is true -->
    <ListView ItemsSource="{x:Bind ViewModel.AdvancedOptions}"/>
</Grid>

<!-- Static deferred loading (realized via FindName) -->
<Grid x:Name="OnboardingPanel" x:Load="False">
    <StackPanel>
        <TextBlock Text="Welcome!" Style="{StaticResource TitleTextBlockStyle}"/>
        <!-- Onboarding content -->
    </StackPanel>
</Grid>
```

```csharp
// Realize deferred element
private void ShowOnboarding()
{
    FindName("OnboardingPanel"); // Creates the element
}

// Unload element to release memory
private void HideOnboarding()
{
    UnloadObject(OnboardingPanel); // Removes from tree and releases memory
}
```

**x:Load Behavior**:
- `x:Load="False"`: Element not created, placeholder marks position (~600 bytes)
- `x:Load="True"` or binding returns true: Element created and added to visual tree
- `UnloadObject()`: Removes element and releases memory

**Common Mistakes**:
```xml
<!-- WRONG: Visibility.Collapsed still creates element -->
<Grid Visibility="Collapsed">
    <ExpensiveControl/>  <!-- Created but hidden - wastes memory -->
</Grid>
```

**Uno Platform Notes**: x:Load behavior was aligned with WinUI in Uno Platform 5.6. Changing Visibility no longer automatically affects x:Load state.

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/xaml-platform/x-load-attribute

---

## XAML-002: x:DeferLoadStrategy is Deprecated

**Rule**: Use `x:Load` instead of `x:DeferLoadStrategy`. The latter is superseded as of Windows 10 version 1703.

**Why**: `x:Load` provides the same deferred loading capability plus the ability to unload elements. `x:DeferLoadStrategy` is legacy and lacks unload support.

**Example (Correct - x:Load)**:
```xml
<Grid x:Name="DeferredContent" x:Load="{x:Bind ShowContent, Mode=OneWay}">
    <!-- Content deferred until ShowContent is true -->
</Grid>
```

**Deprecated (x:DeferLoadStrategy)**:
```xml
<!-- DEPRECATED: Don't use x:DeferLoadStrategy in new code -->
<Grid x:Name="DeferredContent" x:DeferLoadStrategy="Lazy">
    <!-- Use x:Load instead -->
</Grid>
```

**Migration**:
| Old | New |
|-----|-----|
| `x:DeferLoadStrategy="Lazy"` | `x:Load="False"` |
| `FindName("ElementName")` | `FindName("ElementName")` (same) |
| N/A | `UnloadObject(element)` (new capability) |

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/xaml-platform/x-deferloadstrategy-attribute

---

## XAML-003: Defer Tab/Pivot Content

**Rule**: Use x:Load on non-default tabs in TabView, Pivot, or NavigationView to defer their content creation.

**Why**: Users often view only the first tab. Creating all tab content upfront wastes startup time and memory for content that may never be viewed.

**Example (Correct)**:
```xml
<TabView SelectionChanged="TabView_SelectionChanged">
    <TabViewItem Header="Overview">
        <!-- First tab - load immediately -->
        <local:OverviewContent/>
    </TabViewItem>

    <TabViewItem Header="Details">
        <Grid x:Name="DetailsContent" x:Load="False">
            <!-- Deferred until tab selected -->
            <local:DetailsView/>
        </Grid>
    </TabViewItem>

    <TabViewItem Header="Settings">
        <Grid x:Name="SettingsContent" x:Load="False">
            <!-- Deferred until tab selected -->
            <local:SettingsView/>
        </Grid>
    </TabViewItem>
</TabView>
```

```csharp
private void TabView_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    var tabView = sender as TabView;
    var index = tabView.SelectedIndex;

    // Realize content for selected tab
    switch (index)
    {
        case 1: FindName("DetailsContent"); break;
        case 2: FindName("SettingsContent"); break;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-startup-time

---

## XAML-004: Keep Visual Tree Minimal at Startup

**Rule**: Only include essential UI in initial page load. Defer secondary content, dialogs, and rarely-used panels.

**Why**: Every element in the visual tree adds to startup time. Deferring non-essential UI improves perceived performance.

**Startup Optimization Checklist**:
1. Above-the-fold content loads first
2. Loading indicators show immediately
3. Secondary panels use x:Load
4. Dialogs created on-demand
5. Heavy controls (maps, WebView) deferred

**Example (Correct)**:
```xml
<Grid>
    <!-- Essential: Shows immediately -->
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <TextBlock Text="Welcome" Style="{StaticResource TitleTextBlockStyle}"/>

    <!-- Loading indicator shows while data loads -->
    <ProgressRing Grid.Row="1"
                  IsActive="{x:Bind ViewModel.IsLoading, Mode=OneWay}"
                  HorizontalAlignment="Center"/>

    <!-- Main content (shown after data loads) -->
    <ListView Grid.Row="1"
              Visibility="{x:Bind ViewModel.HasData, Mode=OneWay}"
              ItemsSource="{x:Bind ViewModel.Items}"/>

    <!-- Deferred: Filter panel -->
    <Grid x:Name="FilterPanel" x:Load="False" Grid.RowSpan="2">
        <local:FilterControl/>
    </Grid>

    <!-- Deferred: Help overlay -->
    <Grid x:Name="HelpOverlay" x:Load="False" Grid.RowSpan="2">
        <local:HelpContent/>
    </Grid>
</Grid>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-startup-time

---

## XAML-005: x:Load Restrictions

**Rule**: Be aware of x:Load limitations: requires x:Name, can't use on root elements, can't use in ResourceDictionary, affects ElementName bindings.

**Why**: Violating these restrictions causes compile errors or runtime issues. Understanding limitations helps design appropriate solutions.

**Restrictions**:
| Restriction | Reason |
|-------------|--------|
| Requires `x:Name` | Needed to reference element for realization |
| Not on root elements | Page/UserControl/DataTemplate roots must exist |
| Not in ResourceDictionary | Resources are shared, can't be individually loaded |
| Can't be ElementName target | Element may not exist when binding evaluates |

**Example - ElementName Binding Issue**:
```xml
<!-- WRONG: ElementName binding to deferred element -->
<TextBlock Text="{Binding ElementName=DeferredSlider, Path=Value}"/>
<Slider x:Name="DeferredSlider" x:Load="{x:Bind ShowSlider}"/>
<!-- Binding fails if ShowSlider is false -->

<!-- CORRECT: Use x:Bind which handles deferred elements -->
<TextBlock Text="{x:Bind SliderValue, Mode=OneWay}"/>
<Slider x:Name="DeferredSlider"
        x:Load="{x:Bind ShowSlider}"
        Value="{x:Bind SliderValue, Mode=TwoWay}"/>
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/xaml-platform/x-load-attribute

---

## XAML-006: Use x:Phase for Phased Template Loading

**Rule**: In DataTemplates, use `x:Phase` to prioritize which elements render first during rapid scrolling.

**Why**: During fast scrolling, showing essential content first (text) before expensive content (images) keeps scrolling smooth.

**Example (Correct)**:
```xml
<DataTemplate x:DataType="local:Product">
    <Grid Height="80" Padding="12">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="60"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <!-- Phase 0 (default): Placeholder shows first -->
        <Border Background="{ThemeResource ControlFillColorSecondaryBrush}"/>

        <!-- Phase 1: Image loads after initial render -->
        <Image x:Phase="1"
               Source="{x:Bind ImageUrl}"
               Stretch="UniformToFill"/>

        <StackPanel Grid.Column="1" Margin="12,0,0,0">
            <!-- Phase 0: Title shows immediately -->
            <TextBlock Text="{x:Bind Name}"
                       Style="{StaticResource BodyStrongTextBlockStyle}"/>

            <!-- Phase 1: Description loads after -->
            <TextBlock x:Phase="1"
                       Text="{x:Bind Description}"
                       Foreground="{ThemeResource TextFillColorSecondaryBrush}"
                       MaxLines="2"/>

            <!-- Phase 2: Price loads last -->
            <TextBlock x:Phase="2"
                       Text="{x:Bind FormattedPrice}"
                       Foreground="{ThemeResource AccentTextFillColorPrimaryBrush}"/>
        </StackPanel>
    </Grid>
</DataTemplate>
```

**Phase Guidelines**:
- Phase 0: Essential identifying content (title, name)
- Phase 1: Important secondary content (images, descriptions)
- Phase 2+: Supplementary content (prices, dates, actions)

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/xaml-platform/x-phase-attribute

---

## XAML-007: Minimize Merged Dictionary Count

**Rule**: Limit the number of merged ResourceDictionaries. Combine related resources into single files.

**Why**: Each merged dictionary adds startup overhead. XAML parser must load and merge each file. Too many small dictionaries slow startup.

**Example (Correct - Consolidated)**:
```xml
<!-- App.xaml - Few merged dictionaries -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <XamlControlsResources/>
            <ResourceDictionary Source="Themes/AppTheme.xaml"/>
            <ResourceDictionary Source="Styles/ControlStyles.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

**Common Mistakes**:
```xml
<!-- WRONG: Too many small dictionaries -->
<ResourceDictionary.MergedDictionaries>
    <ResourceDictionary Source="Styles/ButtonStyles.xaml"/>
    <ResourceDictionary Source="Styles/TextBlockStyles.xaml"/>
    <ResourceDictionary Source="Styles/BorderStyles.xaml"/>
    <ResourceDictionary Source="Styles/GridStyles.xaml"/>
    <ResourceDictionary Source="Styles/ListViewStyles.xaml"/>
    <ResourceDictionary Source="Colors/LightColors.xaml"/>
    <ResourceDictionary Source="Colors/DarkColors.xaml"/>
    <!-- 20+ more small files... -->
</ResourceDictionary.MergedDictionaries>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-startup-time
