# 03 — Container Controls: Card, Chip, ShadowContainer, ZoomContentControl, Drawer

**Impact**: HIGH — Card, Chip, and ShadowContainer are core container patterns. Wrong control choice or misconfigured properties cause invisible content, missing shadows, or broken interactions.

---

## Card

**Impact**: HIGH — predefined slot-based container following Material Design card layout. Use `Card` when your layout fits the standard slots; use `CardContentControl` for fully custom layouts.

### Card Slots

| Property | Type | Purpose |
|----------|------|---------|
| `HeaderContent` | `object` | Primary title area (top) |
| `SubHeaderContent` | `object` | Subtitle below header |
| `AvatarContent` | `object` | Leading avatar/icon (left of header) |
| `MediaContent` | `object` | Full-width media area (image, video) |
| `SupportingContent` | `object` | Body text / description |
| `IconsContent` | `object` | Trailing action icons (right of header area) |

### Card Styles

| Style Key | Appearance |
|-----------|-----------|
| `ElevatedCardStyle` | Shadow elevation, surface background |
| `FilledCardStyle` | Solid surface-variant background, no elevation |
| `OutlinedCardStyle` | Border outline, transparent background |

### CORRECT Usage

```xml
<!-- Elevated Card with all slots -->
<utu:Card Style="{StaticResource ElevatedCardStyle}">
    <utu:Card.AvatarContent>
        <PersonPicture DisplayName="John Doe" Width="40" Height="40" />
    </utu:Card.AvatarContent>
    <utu:Card.HeaderContent>
        <TextBlock Text="John Doe" Style="{StaticResource TitleMedium}" />
    </utu:Card.HeaderContent>
    <utu:Card.SubHeaderContent>
        <TextBlock Text="Product Designer" Style="{StaticResource BodyMedium}" />
    </utu:Card.SubHeaderContent>
    <utu:Card.IconsContent>
        <Button Style="{StaticResource IconButtonStyle}">
            <SymbolIcon Symbol="More" />
        </Button>
    </utu:Card.IconsContent>
    <utu:Card.MediaContent>
        <Image Source="ms-appx:///Assets/cover.jpg" Stretch="UniformToFill" Height="200" />
    </utu:Card.MediaContent>
    <utu:Card.SupportingContent>
        <TextBlock Text="A brief description of the card content."
                   Style="{StaticResource BodyMedium}"
                   TextWrapping="Wrap" />
    </utu:Card.SupportingContent>
</utu:Card>
```

```xml
<!-- Minimal outlined card -->
<utu:Card Style="{StaticResource OutlinedCardStyle}"
          HeaderContent="Settings"
          SubHeaderContent="Manage your preferences"
          SupportingContent="Tap to configure notification preferences." />
```

### WRONG Usage

```xml
<!-- WRONG: Using Card.Content for custom layout — use CardContentControl instead -->
<utu:Card Style="{StaticResource ElevatedCardStyle}">
    <Grid ColumnDefinitions="Auto,*">
        <Image Source="photo.jpg" Width="80" />
        <TextBlock Text="Custom layout" Grid.Column="1" />
    </Grid>
</utu:Card>
```

---

## CardContentControl

**Impact**: MEDIUM — fully custom card container. Use when the predefined Card slots do not fit your design.

### Usage

```xml
<!-- CORRECT: CardContentControl for custom card layouts -->
<utu:CardContentControl Style="{StaticResource ElevatedCardContentControlStyle}">
    <Grid ColumnDefinitions="80,*" Padding="12" ColumnSpacing="12">
        <Image Source="ms-appx:///Assets/product.jpg"
               Width="80" Height="80"
               Stretch="UniformToFill" />
        <utu:AutoLayout Grid.Column="1" Orientation="Vertical" Spacing="4">
            <TextBlock Text="Product Name" Style="{StaticResource TitleSmall}" />
            <TextBlock Text="$29.99" Style="{StaticResource BodyMedium}" />
            <TextBlock Text="In stock" Foreground="{ThemeResource TertiaryBrush}"
                       Style="{StaticResource LabelMedium}" />
        </utu:AutoLayout>
    </Grid>
</utu:CardContentControl>
```

### CardContentControl Styles

| Style Key | Appearance |
|-----------|-----------|
| `ElevatedCardContentControlStyle` | Shadow elevation |
| `FilledCardContentControlStyle` | Solid fill |
| `OutlinedCardContentControlStyle` | Border outline |

### Card vs CardContentControl Decision

| Use Card when... | Use CardContentControl when... |
|------------------|-------------------------------|
| Layout fits header/sub/avatar/media/supporting slots | You need a completely custom internal layout |
| You want Material Design card conventions | You want card styling (elevation, border) with custom content |
| Quick prototyping | Complex multi-column or non-standard layouts |

---

## Chip and ChipGroup

**Impact**: MEDIUM — selection, filtering, and action triggers. Chips appear in groups with selection behavior.

### Chip Styles

| Style Key | Appearance | Purpose |
|-----------|-----------|---------|
| `AssistChipStyle` | Outlined, no checkmark | Smart actions, suggestions |
| `InputChipStyle` | Outlined with close button | User-entered tags, selections |
| `FilterChipStyle` | Checkmark when selected | Filter categories |
| `SuggestionChipStyle` | Rounded, lighter | Dynamic suggestions |

### ChipGroup Properties

| Property | Type | Values | Purpose |
|----------|------|--------|---------|
| `SelectionMode` | `ChipSelectionMode` | `None`, `Single`, `Multiple` | How many chips can be selected |
| `CanRemove` | `bool` | `True`, `False` | Whether chips show a remove button |
| `ItemsSource` | `IEnumerable` | Collection | Bind chips to data |
| `ItemTemplate` | `DataTemplate` | Template | Template for each chip |
| `SelectedItem` | `object` | Single selection | Currently selected chip (Single mode) |
| `SelectedItems` | `IList<object>` | Multiple selection | Currently selected chips (Multiple mode) |

### Basic Chip Usage

```xml
<!-- CORRECT: Filter chips with multi-select -->
<utu:ChipGroup SelectionMode="Multiple"
               Style="{StaticResource FilterChipGroupStyle}">
    <utu:Chip Content="Small" Style="{StaticResource FilterChipStyle}" />
    <utu:Chip Content="Medium" Style="{StaticResource FilterChipStyle}" />
    <utu:Chip Content="Large" Style="{StaticResource FilterChipStyle}" />
</utu:ChipGroup>
```

```xml
<!-- Data-bound ChipGroup -->
<utu:ChipGroup ItemsSource="{Binding Categories}"
               SelectionMode="Single"
               SelectedItem="{Binding SelectedCategory, Mode=TwoWay}"
               Style="{StaticResource FilterChipGroupStyle}">
    <utu:ChipGroup.ItemTemplate>
        <DataTemplate>
            <utu:Chip Content="{Binding Name}"
                      Style="{StaticResource FilterChipStyle}" />
        </DataTemplate>
    </utu:ChipGroup.ItemTemplate>
</utu:ChipGroup>
```

### Chip with Icon

```xml
<utu:Chip Content="Add Label"
          Style="{StaticResource AssistChipStyle}">
    <utu:Chip.Icon>
        <SymbolIcon Symbol="Add" />
    </utu:Chip.Icon>
</utu:Chip>
```

### Removable Input Chips

```xml
<utu:ChipGroup CanRemove="True" SelectionMode="None">
    <utu:Chip Content="Tag 1" Style="{StaticResource InputChipStyle}" />
    <utu:Chip Content="Tag 2" Style="{StaticResource InputChipStyle}" />
</utu:ChipGroup>
```

### WRONG Usage

```xml
<!-- WRONG: Chip without a style — renders unstyled -->
<utu:Chip Content="Filter" />

<!-- WRONG: Using CheckBox instead of FilterChip for category filters -->
<StackPanel>
    <CheckBox Content="Small" />
    <CheckBox Content="Medium" />
</StackPanel>
```

---

## ShadowContainer

**Impact**: HIGH — adds Material-style shadows (drop and inner) to any control. Critical rule: `Background` MUST be on the `ShadowContainer`, not the child.

### Core Properties

| Property | Type | Purpose |
|----------|------|---------|
| `Background` | `Brush` | **REQUIRED on container** — needed for shadow rendering |
| `CornerRadius` | `CornerRadius` | Rounds the shadow shape |
| `Shadows` | `ShadowCollection` | Collection of `Shadow` objects |

### Shadow Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `OffsetX` | `double` | `0` | Horizontal offset |
| `OffsetY` | `double` | `0` | Vertical offset |
| `BlurRadius` | `double` | `0` | Blur spread |
| `Spread` | `double` | `0` | Size increase beyond element bounds |
| `Color` | `Color` | `Black` | Shadow color |
| `Opacity` | `double` | `1` | Shadow transparency (0-1) |
| `IsInner` | `bool` | `False` | Inner (inset) shadow |

### Drop Shadow

```xml
<!-- CORRECT: Background on ShadowContainer -->
<utu:ShadowContainer Background="{ThemeResource SurfaceBrush}"
                     CornerRadius="12">
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow OffsetX="0" OffsetY="2" BlurRadius="6"
                        Opacity="0.15" Color="Black" />
            <utu:Shadow OffsetX="0" OffsetY="6" BlurRadius="16"
                        Opacity="0.10" Color="Black" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Grid Padding="16">
        <TextBlock Text="Shadowed card" Style="{StaticResource BodyLarge}" />
    </Grid>
</utu:ShadowContainer>
```

### Inner (Inset) Shadow

```xml
<!-- CORRECT: Inner shadow — Background MUST be on ShadowContainer -->
<utu:ShadowContainer Background="{ThemeResource SurfaceVariantBrush}"
                     CornerRadius="8">
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow OffsetX="0" OffsetY="2" BlurRadius="4"
                        Opacity="0.2" Color="Black" IsInner="True" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Grid Padding="12">
        <TextBlock Text="Inset shadow effect" />
    </Grid>
</utu:ShadowContainer>
```

```xml
<!-- WRONG: Background on child — inner shadow WILL NOT render -->
<utu:ShadowContainer CornerRadius="8">
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow OffsetX="0" OffsetY="2" BlurRadius="4"
                        Opacity="0.2" Color="Black" IsInner="True" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Grid Background="{ThemeResource SurfaceVariantBrush}" Padding="12">
        <TextBlock Text="Inner shadow is invisible here" />
    </Grid>
</utu:ShadowContainer>
```

### Layered Shadows (Realistic Depth)

```xml
<!-- Multiple shadows for realistic Material elevation -->
<utu:ShadowContainer Background="{ThemeResource SurfaceBrush}"
                     CornerRadius="16">
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <!-- Ambient shadow (wide, soft) -->
            <utu:Shadow OffsetX="0" OffsetY="1" BlurRadius="3"
                        Spread="1" Opacity="0.12" Color="Black" />
            <!-- Key shadow (directional, sharper) -->
            <utu:Shadow OffsetX="0" OffsetY="4" BlurRadius="8"
                        Spread="0" Opacity="0.18" Color="Black" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Grid Padding="24">
        <TextBlock Text="Elevated content" />
    </Grid>
</utu:ShadowContainer>
```

---

## ZoomContentControl

**Impact**: MEDIUM — zoomable and pannable content area for images, maps, documents, or diagrams.

### Core Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `ZoomLevel` | `double` | `1.0` | Current zoom level |
| `MinZoomLevel` | `double` | `0.1` | Minimum zoom |
| `MaxZoomLevel` | `double` | `10.0` | Maximum zoom |
| `IsZoomAllowed` | `bool` | `True` | Enable/disable zoom |
| `IsActive` | `bool` | `True` | Enable/disable all interaction |
| `AutoFitToCanvas` | `bool` | `False` | Fit content to container on load |
| `IsDoubleTapEnabled` | `bool` | `True` | Double-tap/click to toggle zoom |
| `HorizontalScrollMode` | `ScrollMode` | `Enabled` | Horizontal pan control |
| `VerticalScrollMode` | `ScrollMode` | `Enabled` | Vertical pan control |

### Gesture Support

| Gesture | Action |
|---------|--------|
| Pinch (touch) | Zoom in/out |
| Mouse wheel | Zoom in/out |
| Double-tap / double-click | Toggle between fit and max zoom |
| Drag / pan | Move content within zoomed view |

### Usage

```xml
<!-- CORRECT: Zoomable image viewer -->
<utu:ZoomContentControl MinZoomLevel="0.5"
                        MaxZoomLevel="5.0"
                        AutoFitToCanvas="True"
                        IsDoubleTapEnabled="True">
    <Image Source="ms-appx:///Assets/large-image.jpg"
           Stretch="Uniform" />
</utu:ZoomContentControl>
```

```xml
<!-- Programmatic zoom control -->
<utu:ZoomContentControl x:Name="ZoomViewer"
                        ZoomLevel="{Binding CurrentZoom, Mode=TwoWay}"
                        MinZoomLevel="1.0"
                        MaxZoomLevel="8.0">
    <Canvas Width="2000" Height="2000">
        <!-- Drawing content -->
    </Canvas>
</utu:ZoomContentControl>
```

---

## DrawerControl

**Impact**: MEDIUM — swipe-gesture navigation drawer that slides content in from any edge.

### Core Properties

| Property | Type | Values | Purpose |
|----------|------|--------|---------|
| `OpenDirection` | `DrawerOpenDirection` | `Left`, `Right`, `Up`, `Down` | Edge the drawer slides from |
| `IsOpen` | `bool` | `True`, `False` | Open/close state (bindable) |
| `DrawerLength` | `double` | Pixel value | Width/height of the drawer pane |
| `IsLightDismissEnabled` | `bool` | `True`, `False` | Close on outside tap |
| `DrawerContent` | `object` | Any UI content | Content inside the drawer |
| `EdgeSwipeDetectionLength` | `double` | Pixel value | Width of the swipe detection zone |

### Usage

```xml
<!-- CORRECT: Left drawer with navigation content -->
<utu:DrawerControl OpenDirection="Left"
                   DrawerLength="280"
                   IsLightDismissEnabled="True"
                   IsOpen="{Binding IsMenuOpen, Mode=TwoWay}">
    <utu:DrawerControl.DrawerContent>
        <utu:AutoLayout Orientation="Vertical" Spacing="4" Padding="16">
            <TextBlock Text="Menu" Style="{StaticResource TitleMedium}" />
            <Button Content="Home" Style="{StaticResource TextButtonStyle}" />
            <Button Content="Profile" Style="{StaticResource TextButtonStyle}" />
            <Button Content="Settings" Style="{StaticResource TextButtonStyle}" />
        </utu:AutoLayout>
    </utu:DrawerControl.DrawerContent>

    <!-- Main content -->
    <Grid>
        <TextBlock Text="Main page content" />
    </Grid>
</utu:DrawerControl>
```

```xml
<!-- Bottom drawer for detail panel -->
<utu:DrawerControl OpenDirection="Up"
                   DrawerLength="400"
                   IsLightDismissEnabled="True"
                   IsOpen="{Binding IsDetailOpen, Mode=TwoWay}">
    <utu:DrawerControl.DrawerContent>
        <ScrollViewer>
            <utu:AutoLayout Orientation="Vertical" Spacing="8" Padding="16">
                <TextBlock Text="Details" Style="{StaticResource TitleLarge}" />
                <TextBlock Text="{Binding DetailText}" TextWrapping="Wrap" />
            </utu:AutoLayout>
        </ScrollViewer>
    </utu:DrawerControl.DrawerContent>

    <Grid>
        <ListView ItemsSource="{Binding Items}" />
    </Grid>
</utu:DrawerControl>
```

---

## DrawerFlyoutPresenter

**Impact**: MEDIUM — presents a `Flyout` as a drawer (bottom sheet, side panel). Applied as a `FlyoutPresenterStyle` on a standard `Flyout`.

### Styles

| Style Key | Behavior |
|-----------|---------|
| `BottomDrawerFlyoutPresenterStyle` | Slides up from bottom (mobile bottom sheet) |
| `TopDrawerFlyoutPresenterStyle` | Slides down from top |
| `LeftDrawerFlyoutPresenterStyle` | Slides in from left |
| `RightDrawerFlyoutPresenterStyle` | Slides in from right |

### Usage

```xml
<!-- CORRECT: Bottom sheet flyout -->
<Button Content="Show Details">
    <Button.Flyout>
        <Flyout FlyoutPresenterStyle="{StaticResource BottomDrawerFlyoutPresenterStyle}">
            <utu:AutoLayout Orientation="Vertical" Spacing="12" Padding="16">
                <TextBlock Text="Bottom Sheet" Style="{StaticResource TitleLarge}" />
                <TextBlock Text="Swipe down or tap outside to dismiss."
                           Style="{StaticResource BodyMedium}" TextWrapping="Wrap" />
                <Button Content="Confirm" Style="{StaticResource FilledButtonStyle}" />
            </utu:AutoLayout>
        </Flyout>
    </Button.Flyout>
</Button>
```

```xml
<!-- Right-side drawer flyout for settings -->
<Button Content="Settings">
    <Button.Flyout>
        <Flyout FlyoutPresenterStyle="{StaticResource RightDrawerFlyoutPresenterStyle}">
            <Grid Width="320" Padding="16">
                <utu:AutoLayout Orientation="Vertical" Spacing="8">
                    <TextBlock Text="Settings" Style="{StaticResource TitleMedium}" />
                    <ToggleSwitch Header="Dark Mode" />
                    <ToggleSwitch Header="Notifications" />
                </utu:AutoLayout>
            </Grid>
        </Flyout>
    </Button.Flyout>
</Button>
```

### DrawerFlyoutPresenter vs DrawerControl

| Use DrawerFlyoutPresenter when... | Use DrawerControl when... |
|-----------------------------------|--------------------------|
| Content is transient (action sheet, picker) | Drawer is persistent part of page layout |
| Triggered by a button/action | Swipe-to-reveal is the primary interaction |
| Standard flyout dismiss behavior needed | Need programmatic open/close control |
| Bottom sheet pattern on mobile | Side navigation panel |

---

## Common Mistakes

| Mistake | Impact | Symptom | Fix |
|---------|--------|---------|-----|
| Background on child instead of ShadowContainer | CRITICAL | Inner shadows invisible | Move `Background` to `ShadowContainer` |
| Card used for custom layouts | MEDIUM | Fighting predefined slot template | Use `CardContentControl` for custom layouts |
| Chip without a style | HIGH | Chip renders unstyled/invisible | Apply `AssistChipStyle`, `FilterChipStyle`, etc. |
| ChipGroup without `SelectionMode` | MEDIUM | Chips not selectable | Set `SelectionMode="Single"` or `Multiple` |
| DrawerControl without `DrawerLength` | MEDIUM | Drawer has no visible size | Set explicit `DrawerLength` |
| ShadowContainer without `CornerRadius` | LOW | Shadow shape doesn't match child | Set matching `CornerRadius` on ShadowContainer |
| ZoomContentControl without `AutoFitToCanvas` | LOW | Content appears at 1:1 pixel size on load | Set `AutoFitToCanvas="True"` for fit-to-view on load |

---

## Reference Links

- [Card Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/Card.html)
- [CardContentControl Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/CardContentControl.html)
- [Chip and ChipGroup Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ChipAndChipGroup.html)
- [ShadowContainer Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ShadowContainer.html)
- [ZoomContentControl Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/ZoomContentControl.html)
- [DrawerControl Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/DrawerControl.html)
- [DrawerFlyoutPresenter Documentation](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/controls/DrawerFlyoutPresenter.html)
