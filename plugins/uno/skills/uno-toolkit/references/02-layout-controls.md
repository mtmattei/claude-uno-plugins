# 02 — Layout Controls: AutoLayout, SafeArea, Divider, Responsive

**Impact**: HIGH — AutoLayout is the primary layout primitive for Toolkit apps. SafeArea is mandatory for mobile form pages. Missing either causes broken layouts or hidden content.

---

## AutoLayout

**Impact**: CRITICAL — Figma-aligned layout control that replaces StackPanel with proper spacing, alignment, and stretch behavior.

### Core Properties

| Property | Type | Values | Default | Purpose |
|----------|------|--------|---------|---------|
| `Orientation` | `Orientation` | `Horizontal`, `Vertical` | `Vertical` | Primary axis direction |
| `Spacing` | `double` | Any positive value | `0` | Gap between children along primary axis |
| `Justify` | `AutoLayoutJustify` | `Stack`, `SpaceBetween` | `Stack` | Distribute children along primary axis |
| `PrimaryAxisAlignment` | `AutoLayoutAlignment` | `Start`, `Center`, `End`, `Stretch` | `Start` | Align children group on primary axis |
| `CounterAxisAlignment` | `AutoLayoutAlignment` | `Start`, `Center`, `End`, `Stretch` | `Start` | Align children on cross axis |
| `Padding` | `Thickness` | Standard thickness | `0` | Inner padding (uniform or per-side) |
| `IsReverseZOrder` | `bool` | `True`, `False` | `False` | Reverse child rendering order |

### Basic Usage

```xml
<!-- CORRECT: Vertical form layout with spacing and cross-axis stretch -->
<utu:AutoLayout Orientation="Vertical"
                Spacing="16"
                PrimaryAxisAlignment="Start"
                CounterAxisAlignment="Stretch"
                Padding="24">
    <TextBlock Text="Sign In" Style="{StaticResource HeadlineMedium}" />
    <TextBox Header="Email" />
    <PasswordBox Header="Password" />
    <Button Content="Sign In" Style="{StaticResource FilledButtonStyle}" />
</utu:AutoLayout>
```

```xml
<!-- CORRECT: Horizontal button row with space-between -->
<utu:AutoLayout Orientation="Horizontal"
                Justify="SpaceBetween"
                CounterAxisAlignment="Center"
                Padding="16">
    <Button Content="Cancel" Style="{StaticResource TextButtonStyle}" />
    <Button Content="Submit" Style="{StaticResource FilledButtonStyle}" />
</utu:AutoLayout>
```

### Per-Child Attached Properties

AutoLayout supports attached properties on individual children for fine-grained control:

| Attached Property | Type | Values | Purpose |
|-------------------|------|--------|---------|
| `utu:AutoLayout.PrimaryAlignment` | `AutoLayoutPrimaryAlignment` | `Auto`, `Stretch` | Override child's primary axis behavior — `Stretch` fills remaining space |
| `utu:AutoLayout.CounterAlignment` | `AutoLayoutAlignment` | `Start`, `Center`, `End`, `Stretch` | Override child's cross-axis alignment |
| `utu:AutoLayout.CounterLength` | `double` | Fixed pixel value | Set explicit cross-axis size for a child |
| `utu:AutoLayout.IsIndependentLayout` | `bool` | `True`, `False` | Remove child from flow (absolute positioning relative to AutoLayout) |
| `utu:AutoLayout.PrimaryLength` | `double` | Fixed pixel value | Set explicit primary-axis size for a child |

### Stretch a Child to Fill Remaining Space

```xml
<!-- CORRECT: middle child stretches to fill available space -->
<utu:AutoLayout Orientation="Vertical" Spacing="8" Padding="16">
    <TextBlock Text="Header" Style="{StaticResource TitleLarge}" />

    <!-- This fills remaining vertical space -->
    <ScrollViewer utu:AutoLayout.PrimaryAlignment="Stretch">
        <TextBlock Text="Scrollable content..." TextWrapping="Wrap" />
    </ScrollViewer>

    <Button Content="Action"
            Style="{StaticResource FilledButtonStyle}"
            HorizontalAlignment="Stretch" />
</utu:AutoLayout>
```

```xml
<!-- WRONG: using VerticalAlignment="Stretch" on StackPanel child — ignored -->
<StackPanel Orientation="Vertical">
    <TextBlock Text="Header" />
    <ScrollViewer VerticalAlignment="Stretch">  <!-- Does nothing in StackPanel -->
        <TextBlock Text="Content..." />
    </ScrollViewer>
    <Button Content="Action" />
</StackPanel>
```

### Independent Layout (Absolute Positioning)

```xml
<!-- Overlay badge on top-right corner -->
<utu:AutoLayout Orientation="Vertical" Spacing="8">
    <Image Source="photo.jpg" Width="200" Height="200" />
    <TextBlock Text="Caption" />

    <!-- Positioned independently from flow -->
    <Border utu:AutoLayout.IsIndependentLayout="True"
            HorizontalAlignment="Right"
            VerticalAlignment="Top"
            Background="Red" CornerRadius="10"
            Padding="4,2">
        <TextBlock Text="NEW" Foreground="White" Style="{StaticResource LabelSmall}" />
    </Border>
</utu:AutoLayout>
```

### AutoLayout vs StackPanel

| Feature | AutoLayout | StackPanel |
|---------|-----------|------------|
| Spacing between children | `Spacing` property | Manual `Margin` on each child |
| Child stretch on primary axis | `PrimaryAlignment="Stretch"` | Not supported |
| Cross-axis alignment | `CounterAxisAlignment` | Individual `HorizontalAlignment` / `VerticalAlignment` |
| Space distribution | `Justify="SpaceBetween"` | Not supported |
| Absolute overlay children | `IsIndependentLayout="True"` | Not supported (use Grid) |
| Figma Auto Layout parity | Full | None |

---

## SafeArea

**Impact**: CRITICAL — required on mobile to prevent content from being hidden behind notches, status bars, navigation bars, and on-screen keyboards.

### Core Properties

| Property | Type | Values | Purpose |
|----------|------|--------|---------|
| `Insets` | `SafeAreaInsets` | `VisibleBounds`, `SoftInput`, `VisibleBounds,SoftInput` | Which system areas to avoid |
| `Mode` | `SafeAreaMode` | `Padding`, `Margin`, `InsetOverride` | How insets are applied |

### Insets Values

| Value | What It Avoids | When to Use |
|-------|---------------|-------------|
| `VisibleBounds` | Notches, status bar, navigation bar, rounded corners | Always on pages with edge-to-edge content |
| `SoftInput` | On-screen keyboard | Pages with TextBox, PasswordBox, or any input |
| `VisibleBounds,SoftInput` | All of the above | **Recommended default** for most pages |

### Mode Values

| Mode | Behavior | Best For |
|------|----------|----------|
| `Padding` | Adds padding inside the SafeArea control | Most cases — content stays within safe bounds |
| `Margin` | Adds margin outside the SafeArea control | When SafeArea is not the outermost container |
| `InsetOverride` | Overrides the insets of specific properties on the child | Advanced: directly modifies child's Margin or Padding |

### Mobile Form Pattern (Mandatory)

```xml
<!-- CORRECT: SafeArea with SoftInput for keyboard avoidance on forms -->
<utu:SafeArea Insets="VisibleBounds,SoftInput" Mode="Padding">
    <ScrollViewer>
        <utu:AutoLayout Orientation="Vertical" Spacing="12" Padding="16">
            <TextBox Header="First Name" />
            <TextBox Header="Last Name" />
            <TextBox Header="Email" />
            <PasswordBox Header="Password" />
            <Button Content="Create Account"
                    Style="{StaticResource FilledButtonStyle}" />
        </utu:AutoLayout>
    </ScrollViewer>
</utu:SafeArea>
```

```xml
<!-- WRONG: No SafeArea — keyboard covers bottom fields on iOS/Android -->
<ScrollViewer>
    <StackPanel>
        <TextBox Header="First Name" />
        <TextBox Header="Last Name" />
        <TextBox Header="Email" />
        <PasswordBox Header="Password" />
        <Button Content="Create Account" />
    </StackPanel>
</ScrollViewer>
```

### SafeArea on Non-Form Pages

```xml
<!-- CORRECT: VisibleBounds only for pages without text input -->
<utu:SafeArea Insets="VisibleBounds" Mode="Padding">
    <Grid>
        <ListView ItemsSource="{Binding Items}" />
    </Grid>
</utu:SafeArea>
```

### Attached Property Usage

SafeArea can also be applied as an attached property on any control:

```xml
<!-- Attached property syntax on a Grid -->
<Grid utu:SafeArea.Insets="VisibleBounds,SoftInput"
      utu:SafeArea.Mode="Padding">
    <TextBox Header="Search" />
</Grid>
```

---

## Divider

**Impact**: LOW — simple visual separator between content sections.

### Usage

```xml
<!-- Horizontal divider (default) -->
<utu:Divider />

<!-- Divider with subheader text -->
<utu:Divider SubHeader="Section Title" />

<!-- Vertical divider inside horizontal layout -->
<utu:AutoLayout Orientation="Horizontal" Spacing="8">
    <TextBlock Text="Left" />
    <utu:Divider Orientation="Vertical" />
    <TextBlock Text="Right" />
</utu:AutoLayout>
```

### Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `SubHeader` | `string` | `null` | Optional text label |
| `SubHeaderForeground` | `Brush` | Theme default | Text color |
| `Orientation` | `Orientation` | `Horizontal` | Divider direction |

---

## ResponsiveExtension

**Impact**: MEDIUM — adapts property values based on screen width breakpoints. Essential for responsive layouts.

### Default Breakpoints

| Size | Min Width |
|------|-----------|
| `Narrow` | 0px |
| `Normal` | 641px |
| `Wide` | 1008px |

### Usage

```xml
<!-- CORRECT: Change Orientation based on screen width -->
<utu:AutoLayout Orientation="{utu:Responsive Normal=Vertical, Wide=Horizontal}"
                Spacing="{utu:Responsive Normal=8, Wide=16}">
    <TextBlock Text="Item A" />
    <TextBlock Text="Item B" />
</utu:AutoLayout>
```

```xml
<!-- CORRECT: Show/hide elements responsively -->
<Border Visibility="{utu:Responsive Narrow=Collapsed, Normal=Visible}" />
```

```xml
<!-- Responsive Padding -->
<utu:AutoLayout Padding="{utu:Responsive Narrow='8', Normal='16', Wide='24'}">
    <TextBlock Text="Content" />
</utu:AutoLayout>
```

### Custom Breakpoints via ResponsiveLayout

Define custom breakpoints as a resource:

```xml
<Page.Resources>
    <utu:ResponsiveLayout x:Key="CustomBreakpoints"
                          Narrowest="0"
                          Narrow="320"
                          Normal="600"
                          Wide="900"
                          Widest="1200" />
</Page.Resources>

<!-- Reference the custom layout -->
<utu:AutoLayout Orientation="{utu:Responsive Layout={StaticResource CustomBreakpoints},
                              Narrow=Vertical, Wide=Horizontal}">
    <TextBlock Text="Adaptive content" />
</utu:AutoLayout>
```

### ResponsiveLayout Breakpoint Names

| Name | Description |
|------|-------------|
| `Narrowest` | Smallest screens (watches, compact phones) |
| `Narrow` | Phones in portrait |
| `Normal` | Tablets, phones in landscape |
| `Wide` | Desktop, large tablets |
| `Widest` | Ultra-wide, multi-monitor |

---

## ResponsiveView

**Impact**: MEDIUM — switches entire templates based on screen width. Use when layout changes are too complex for property-level `ResponsiveExtension`.

### Usage

```xml
<utu:ResponsiveView>
    <utu:ResponsiveView.NarrowTemplate>
        <DataTemplate>
            <!-- Phone layout: single column -->
            <utu:AutoLayout Orientation="Vertical" Spacing="8">
                <TextBlock Text="Mobile View" />
                <ListView ItemsSource="{Binding Items}" />
            </utu:AutoLayout>
        </DataTemplate>
    </utu:ResponsiveView.NarrowTemplate>

    <utu:ResponsiveView.WideTemplate>
        <DataTemplate>
            <!-- Desktop layout: side-by-side -->
            <Grid ColumnDefinitions="300,*">
                <ListView ItemsSource="{Binding Items}" Grid.Column="0" />
                <ContentControl Content="{Binding SelectedItem}" Grid.Column="1" />
            </Grid>
        </DataTemplate>
    </utu:ResponsiveView.WideTemplate>
</utu:ResponsiveView>
```

### Template Properties

| Property | Breakpoint | Fallback |
|----------|------------|----------|
| `NarrowestTemplate` | Narrowest | None |
| `NarrowTemplate` | Narrow | `NarrowestTemplate` |
| `NormalTemplate` | Normal | `NarrowTemplate` |
| `WideTemplate` | Wide | `NormalTemplate` |
| `WidestTemplate` | Widest | `WideTemplate` |

> Templates cascade: if `NormalTemplate` is not set, `NarrowTemplate` is used. Only define the breakpoints where your layout actually changes.

### Custom Breakpoints on ResponsiveView

```xml
<utu:ResponsiveView ResponsiveLayout="{StaticResource CustomBreakpoints}">
    <utu:ResponsiveView.NarrowTemplate>
        <DataTemplate>...</DataTemplate>
    </utu:ResponsiveView.NarrowTemplate>
</utu:ResponsiveView>
```

---

## Common Mistakes

| Mistake | Impact | Symptom | Fix |
|---------|--------|---------|-----|
| Using StackPanel instead of AutoLayout | MEDIUM | Manual margins everywhere, no stretch | Replace with `AutoLayout` for Spacing, Justify, alignment |
| SafeArea missing `SoftInput` on form pages | CRITICAL | Keyboard covers TextBox on iOS/Android | Add `Insets="VisibleBounds,SoftInput"` |
| SafeArea wrapping a non-scrollable form | HIGH | Content pushed off screen when keyboard opens | Wrap form content in `ScrollViewer` inside SafeArea |
| AutoLayout child with `Margin` instead of `Spacing` | MEDIUM | Inconsistent gaps, Spacing + Margin stack | Use parent `Spacing` only; remove child Margin |
| ResponsiveExtension without all needed breakpoints | LOW | Layout jumps at certain widths | Define values for Narrow, Normal, Wide at minimum |
| AutoLayout without explicit alignment | MEDIUM | Children bunched at Start with no stretch | Set `PrimaryAxisAlignment` and `CounterAxisAlignment` |
| Using `Width`/`Height` on AutoLayout children | MEDIUM | Conflicts with stretch behavior | Use `PrimaryLength`/`CounterLength` attached props or let AutoLayout manage sizes |

---

## Reference Links

- [AutoLayout Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/AutoLayout.html)
- [SafeArea Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/SafeArea.html)
- [Divider Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/Divider.html)
- [ResponsiveExtension Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/responsive-extension.html)
- [ResponsiveView Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ResponsiveView.html)
