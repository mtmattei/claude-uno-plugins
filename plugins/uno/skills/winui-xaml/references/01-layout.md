# Layout Skills - WinUI 3 XAML

## LAYOUT-001: Prefer Grid Over Nested StackPanels

**Rule**: Use a single Grid with row/column definitions instead of nested StackPanels.

**Why**: Grid performs single-pass layout while nested StackPanels require multiple layout passes. Each nested panel adds measure/arrange overhead. Performance impact: 2-4x faster layout for complex forms.

**Example (Correct)**:
```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>

    <TextBlock Text="Name:" Grid.Row="0" Grid.Column="0" VerticalAlignment="Center"/>
    <TextBox Grid.Row="0" Grid.Column="1"/>

    <TextBlock Text="Email:" Grid.Row="1" Grid.Column="0" VerticalAlignment="Center"/>
    <TextBox Grid.Row="1" Grid.Column="1"/>

    <ListView Grid.Row="2" Grid.ColumnSpan="2"/>
</Grid>
```

**Common Mistakes**:
```xml
<!-- WRONG: Nested StackPanels cause multiple layout passes -->
<StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="Name:"/>
        <TextBox/>
    </StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="Email:"/>
        <TextBox/>
    </StackPanel>
</StackPanel>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layout-panels

---

## LAYOUT-002: Keep Visual Tree Shallow

**Rule**: Minimize visual tree depth to 4-6 levels. Remove unnecessary container elements.

**Why**: Each visual tree level adds layout pass time and hit-testing overhead. Deep trees cause exponential performance degradation. Measured impact: 2.5x faster layout/hit-testing with shallow trees.

**Example (Correct)**:
```xml
<!-- 2 levels deep -->
<Grid Padding="16">
    <TextBlock Text="{x:Bind Title}" Style="{StaticResource TitleTextBlockStyle}"/>
</Grid>
```

**Common Mistakes**:
```xml
<!-- WRONG: 6 unnecessary levels -->
<Border>
    <Grid>
        <StackPanel>
            <Border>
                <Grid>
                    <TextBlock Text="{x:Bind Title}"/>
                </Grid>
            </Border>
        </StackPanel>
    </Grid>
</Border>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-xaml-layout

---

## LAYOUT-003: Use Fixed Sizes for Virtualized Items

**Rule**: Set explicit Height on items in virtualized lists (ListView, GridView, ItemsRepeater).

**Why**: Fixed sizes allow the virtualizing panel to skip the measure pass for off-screen items. Auto-sized items require measuring every item. Impact: 4x faster layout for 1000+ item lists.

**Example (Correct)**:
```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <Grid Height="72" Padding="12">
                <TextBlock Text="{x:Bind Name}"/>
            </Grid>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**Common Mistakes**:
```xml
<!-- WRONG: Auto height defeats virtualization optimization -->
<ListView.ItemTemplate>
    <DataTemplate x:DataType="local:Item">
        <StackPanel>  <!-- No fixed height -->
            <TextBlock Text="{x:Bind Name}" TextWrapping="Wrap"/>
            <TextBlock Text="{x:Bind Description}" TextWrapping="Wrap"/>
        </StackPanel>
    </DataTemplate>
</ListView.ItemTemplate>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-gridview-and-listview

---

## LAYOUT-004: Use Canvas for Absolute Positioning

**Rule**: When elements have fixed absolute positions (games, diagrams, overlays), use Canvas instead of Grid with margins.

**Why**: Canvas skips measure/arrange entirely for children with explicit Canvas.Left/Top. Grid with margins still calculates relative positioning. Impact: 5x faster for 100+ positioned elements.

**Example (Correct)**:
```xml
<Canvas Width="800" Height="600">
    <Ellipse Canvas.Left="100" Canvas.Top="50" Width="20" Height="20" Fill="Red"/>
    <Ellipse Canvas.Left="200" Canvas.Top="150" Width="20" Height="20" Fill="Blue"/>
    <Line Canvas.Left="100" Canvas.Top="50" X2="100" Y2="100" Stroke="Black"/>
</Canvas>
```

**Common Mistakes**:
```xml
<!-- WRONG: Using Grid margins for absolute positioning -->
<Grid Width="800" Height="600">
    <Ellipse Margin="100,50,0,0" HorizontalAlignment="Left" VerticalAlignment="Top"
             Width="20" Height="20" Fill="Red"/>
</Grid>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layout-panels#canvas

---

## LAYOUT-005: Limit RelativePanel Constraint Complexity

**Rule**: Use RelativePanel for simple 2-5 element layouts. For complex layouts with many cross-dependencies, prefer Grid.

**Why**: RelativePanel resolves constraints iteratively. Many inter-element dependencies create O(n^2) resolution complexity. Grid's row/column model resolves in O(n).

**Example (Correct - Simple RelativePanel)**:
```xml
<RelativePanel>
    <Image x:Name="Avatar" Width="48" Height="48"/>
    <TextBlock x:Name="Title"
               RelativePanel.RightOf="Avatar"
               RelativePanel.AlignTopWith="Avatar"
               Margin="12,0,0,0"/>
    <TextBlock RelativePanel.RightOf="Avatar"
               RelativePanel.Below="Title"
               Margin="12,4,0,0"/>
</RelativePanel>
```

**Common Mistakes**:
```xml
<!-- WRONG: Complex cross-dependencies in RelativePanel -->
<RelativePanel>
    <Element1 x:Name="E1"/>
    <Element2 x:Name="E2" RelativePanel.RightOf="E1"/>
    <Element3 x:Name="E3" RelativePanel.Below="E2" RelativePanel.AlignLeftWith="E1"/>
    <Element4 RelativePanel.RightOf="E3" RelativePanel.AlignTopWith="E2"
              RelativePanel.LeftOf="E5"/>
    <Element5 x:Name="E5" RelativePanel.AlignRightWithPanel="True"/>
    <!-- Many more interdependent elements... -->
</RelativePanel>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/layout/layout-panels#relativepanel

---

## LAYOUT-006: Use Translation for Position Animations

**Rule**: Animate position using RenderTransform (TranslateTransform) or Composition Offset, never Margin.

**Why**: Margin changes trigger layout passes. Translation/Offset changes run on the composition thread without layout. Impact: 60 FPS vs 15-25 FPS for multi-element animations.

**Example (Correct)**:
```xml
<Button x:Name="SlideButton" Content="Slide Me">
    <Button.RenderTransform>
        <TranslateTransform x:Name="SlideTransform"/>
    </Button.RenderTransform>
</Button>
```

```csharp
// Composition API animation (preferred)
var visual = ElementCompositionPreview.GetElementVisual(SlideButton);
var animation = visual.Compositor.CreateVector3KeyFrameAnimation();
animation.InsertKeyFrame(1.0f, new Vector3(100, 0, 0));
animation.Duration = TimeSpan.FromMilliseconds(300);
visual.StartAnimation("Offset", animation);
```

**Common Mistakes**:
```xml
<!-- WRONG: Animating Margin triggers layout -->
<Storyboard>
    <ThicknessAnimation Storyboard.TargetProperty="Margin"
                        To="100,0,0,0" Duration="0:0:0.3"/>
</Storyboard>
```

**Uno Platform Notes**: Composition animations are supported on Skia backends. On native iOS/Android, use standard XAML animations which are hardware-accelerated.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/motion/motion-in-practice

---

## LAYOUT-007: Use Responsive Layout Techniques

**Rule**: Use declarative responsive techniques instead of code-behind window size handlers. For Uno Platform, prefer `ResponsiveExtension` for property changes and `ResponsiveView` for layout changes. Use `AdaptiveTrigger` for WinUI 3 standard compatibility or when combining responsive states with animations.

**Why**: Declarative approaches work with XAML Hot Reload, are optimized for resize events, and avoid flickering. Uno Toolkit's responsive helpers provide cleaner syntax with predefined breakpoints (Narrowest ≤150px, Narrow ≤300px, Normal ≤600px, Wide ≤800px, Widest ≤1080px).

### Option 1: ResponsiveExtension (Uno Platform - Preferred for Property Changes)

Best for: Changing individual property values based on screen width.

```xml
xmlns:utu="using:Uno.Toolkit.UI"

<!-- Single property - inline and concise -->
<StackPanel Orientation="{utu:Responsive Narrow=Vertical, Wide=Horizontal}">
    <Grid Width="{utu:Responsive Narrow=0, Wide=300}"/>
    <Grid/>
</StackPanel>

<!-- Multiple properties -->
<TextBlock Text="Title"
           FontSize="{utu:Responsive Narrow=16, Normal=20, Wide=24}"
           Margin="{utu:Responsive Narrow='8,4', Wide='16,8'}"/>

<!-- Custom breakpoints -->
<Page.Resources>
    <utu:ResponsiveLayout x:Key="CustomLayout" Narrow="400" Wide="1000"/>
</Page.Resources>

<StackPanel Orientation="{utu:Responsive Layout={StaticResource CustomLayout},
                          Narrow=Vertical, Wide=Horizontal}"/>
```

### Option 2: ResponsiveView (Uno Platform - For Different Layouts)

Best for: Completely different UI structures per breakpoint.

```xml
xmlns:utu="using:Uno.Toolkit.UI"

<utu:ResponsiveView>
    <utu:ResponsiveView.NarrowTemplate>
        <DataTemplate>
            <!-- Mobile: stacked layout -->
            <StackPanel>
                <Image Source="header.png"/>
                <TextBlock Text="Title" Style="{StaticResource TitleTextBlockStyle}"/>
                <TextBlock Text="Content"/>
            </StackPanel>
        </DataTemplate>
    </utu:ResponsiveView.NarrowTemplate>
    <utu:ResponsiveView.WideTemplate>
        <DataTemplate>
            <!-- Desktop: side-by-side layout -->
            <Grid ColumnDefinitions="300,*">
                <Image Source="header.png" Grid.Column="0"/>
                <StackPanel Grid.Column="1">
                    <TextBlock Text="Title" Style="{StaticResource TitleTextBlockStyle}"/>
                    <TextBlock Text="Content"/>
                </StackPanel>
            </Grid>
        </DataTemplate>
    </utu:ResponsiveView.WideTemplate>
</utu:ResponsiveView>
```

### Option 3: AdaptiveTrigger (WinUI 3 Standard)

Best for: Windows-only apps, combining responsive states with Storyboard animations, or when Uno Toolkit isn't available.

```xml
<Grid>
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup>
            <VisualState x:Name="Narrow">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="0"/>
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="ContentPanel.Orientation" Value="Vertical"/>
                    <Setter Target="SidePanel.Visibility" Value="Collapsed"/>
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="Wide">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="720"/>
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="ContentPanel.Orientation" Value="Horizontal"/>
                    <Setter Target="SidePanel.Visibility" Value="Visible"/>
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>

    <StackPanel x:Name="ContentPanel">
        <Grid x:Name="SidePanel" Width="300"/>
        <Grid x:Name="MainContent"/>
    </StackPanel>
</Grid>
```

**Comparison**:

| Approach | Lines of XAML | Best For | Requires |
|----------|---------------|----------|----------|
| ResponsiveExtension | 1 per property | Property tweaks | Uno Toolkit |
| ResponsiveView | Medium | Different layouts | Uno Toolkit |
| AdaptiveTrigger | 15+ | Animations, WinUI standard | Nothing (built-in) |

**Common Mistakes**:
```csharp
// WRONG: Manual size handling in code-behind
private void Page_SizeChanged(object sender, SizeChangedEventArgs e)
{
    if (e.NewSize.Width > 720)
    {
        ContentPanel.Orientation = Orientation.Horizontal;
        SidePanel.Visibility = Visibility.Visible;
    }
    else
    {
        ContentPanel.Orientation = Orientation.Vertical;
        SidePanel.Visibility = Visibility.Collapsed;
    }
}
```

```xml
<!-- WRONG: Using AdaptiveTrigger for simple property changes in Uno Platform apps -->
<!-- Use ResponsiveExtension instead for cleaner syntax -->
```

**Uno Platform Notes**:
- ResponsiveExtension has a UWP limitation: cannot directly use non-string properties. Use `{StaticResource}` references for complex types like brushes.
- ResponsiveView and ResponsiveExtension require `Uno.Toolkit.WinUI` package.
- Only define breakpoints you need - undefined breakpoints fall back to the nearest defined value.

**References**:
- https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/responsive-extension.html
- https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ResponsiveView.html
- https://learn.microsoft.com/en-us/windows/apps/design/layout/layouts-with-xaml#adaptive-layouts-with-visual-states-and-state-triggers

---

## LAYOUT-008: Avoid InvalidateArrange Cascades

**Rule**: Batch multiple property changes that affect layout. Avoid setting Width, Height, Margin in loops.

**Why**: Each layout-affecting property change can trigger InvalidateArrange. Multiple changes cause multiple layout passes. Batch changes together for a single pass.

**Example (Correct)**:
```csharp
// Batch changes - properties are set without intermediate layout
foreach (var item in items)
{
    var element = GetElement(item);
    // These queue layout invalidation but don't execute until next frame
    element.Width = newWidth;
    element.Height = newHeight;
    element.Margin = new Thickness(4);
}
// Single layout pass happens on next render frame
```

**Common Mistakes**:
```csharp
// WRONG: Forcing layout inside loop
foreach (var item in items)
{
    var element = GetElement(item);
    element.Width = newWidth;
    element.UpdateLayout(); // Forces immediate layout pass
    // Subsequent property changes trigger more passes
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-xaml-layout

---

## LAYOUT-009: Use AutoLayout for Flexbox-Style Layouts (Uno Toolkit)

**Rule**: Use `AutoLayout` from Uno Toolkit for linear layouts with consistent spacing. Set `Spacing` and `Padding` on the container - never set margins on children.

**Why**: AutoLayout mirrors Figma's Auto Layout behavior, providing cleaner XAML than StackPanel with margins. It handles spacing uniformly, supports primary/counter axis alignment, and allows per-child size overrides. Eliminates inconsistent margin math.

**Example (Correct)**:
```xml
xmlns:utu="using:Uno.Toolkit.UI"

<!-- Vertical stack with consistent spacing -->
<utu:AutoLayout Orientation="Vertical"
                Spacing="12"
                Padding="16"
                PrimaryAxisAlignment="Start"
                CounterAxisAlignment="Stretch">
    <TextBlock Text="Title" Style="{StaticResource TitleTextBlockStyle}"/>
    <TextBlock Text="Subtitle"/>
    <Button Content="Action"/>
</utu:AutoLayout>

<!-- Horizontal layout with centered children -->
<utu:AutoLayout Orientation="Horizontal"
                Spacing="8"
                Padding="16,12"
                PrimaryAxisAlignment="Center"
                CounterAxisAlignment="Center">
    <Button Content="Cancel"/>
    <Button Content="Save" Style="{StaticResource AccentButtonStyle}"/>
</utu:AutoLayout>

<!-- Child with explicit counter-axis size -->
<utu:AutoLayout Orientation="Horizontal" Spacing="8">
    <Border Background="LightGray"
            utu:AutoLayout.CounterLength="48">
        <TextBlock Text="Fixed 48px height"/>
    </Border>
    <TextBlock Text="Auto height" VerticalAlignment="Center"/>
</utu:AutoLayout>

<!-- Space-between distribution -->
<utu:AutoLayout Orientation="Horizontal"
                Justify="SpaceBetween"
                Padding="16">
    <TextBlock Text="Left"/>
    <TextBlock Text="Right"/>
</utu:AutoLayout>
```

**AutoLayout Properties**:
| Property | Values | Description |
|----------|--------|-------------|
| `Orientation` | `Vertical`, `Horizontal` | Stack direction (primary axis) |
| `Spacing` | Double | Gap between children |
| `Padding` | Thickness | Inner padding |
| `PrimaryAxisAlignment` | `Start`, `Center`, `End`, `Stretch` | Alignment along stack direction |
| `CounterAxisAlignment` | `Start`, `Center`, `End`, `Stretch` | Alignment perpendicular to stack |
| `Justify` | `Start`, `Center`, `End`, `SpaceBetween` | Distribution along primary axis |

**Attached Properties** (per-child):
| Property | Description |
|----------|-------------|
| `AutoLayout.PrimaryLength` | Fixed size on primary axis |
| `AutoLayout.CounterLength` | Fixed size on counter axis |
| `AutoLayout.CounterAlignment` | Override counter alignment for this child |
| `AutoLayout.IsIndependentLayout` | Exclude from AutoLayout flow |

**Common Mistakes**:
```xml
<!-- WRONG: Margins on children - use container Spacing instead -->
<utu:AutoLayout Orientation="Vertical">
    <TextBlock Text="Title" Margin="0,0,0,12"/>
    <TextBlock Text="Subtitle" Margin="0,0,0,12"/>
    <Button Content="Action" Margin="0,0,0,12"/>
</utu:AutoLayout>

<!-- WRONG: Nested AutoLayouts when one would suffice -->
<utu:AutoLayout Orientation="Vertical">
    <utu:AutoLayout Orientation="Horizontal">
        <TextBlock Text="Label"/>
    </utu:AutoLayout>
</utu:AutoLayout>

<!-- WRONG: Using StackPanel with manual spacing -->
<StackPanel>
    <TextBlock Text="Title" Margin="0,0,0,12"/>
    <TextBlock Text="Subtitle" Margin="0,0,0,12"/>
</StackPanel>
```

**Uno Platform Notes**: AutoLayout requires `Uno.Toolkit.UI` package. Add to your project:
```xml
<PackageReference Include="Uno.Toolkit.WinUI" Version="..." />
```

And register in App.xaml:
```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ToolkitResources xmlns="using:Uno.Toolkit.UI"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

**Reference**: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/AutoLayoutControl.html
