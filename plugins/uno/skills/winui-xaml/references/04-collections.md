# Collections & Virtualization Skills - WinUI 3 XAML

## COLLECTIONS-001: Always Use Virtualized Controls for Lists

**Rule**: Use `ListView`, `GridView`, or `ItemsRepeater` (with ScrollViewer) for any collection that could exceed 20 items. Never use `StackPanel` + `ItemsControl` for dynamic lists.

**Why**: Virtualized controls only create UI elements for visible items. Non-virtualized controls create elements for ALL items, causing memory bloat and slow rendering for large collections.

**Example (Correct)**:
```xml
<!-- ListView with virtualization (automatic) -->
<ListView ItemsSource="{x:Bind ViewModel.Items}">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <Grid Height="60" Padding="12">
                <TextBlock Text="{x:Bind Name}"/>
            </Grid>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>

<!-- ItemsRepeater with ScrollViewer (virtualization via layout) -->
<ScrollViewer>
    <ItemsRepeater ItemsSource="{x:Bind ViewModel.Items}"
                   ItemTemplate="{StaticResource ItemTemplate}">
        <ItemsRepeater.Layout>
            <StackLayout Orientation="Vertical" Spacing="4"/>
        </ItemsRepeater.Layout>
    </ItemsRepeater>
</ScrollViewer>
```

**Common Mistakes**:
```xml
<!-- WRONG: ItemsControl has no virtualization -->
<ScrollViewer>
    <ItemsControl ItemsSource="{x:Bind Items}">
        <ItemsControl.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Name}"/>
            </DataTemplate>
        </ItemsControl.ItemTemplate>
    </ItemsControl>
</ScrollViewer>

<!-- WRONG: StackPanel creates all elements -->
<ScrollViewer>
    <StackPanel x:Name="ItemsPanel"/>
</ScrollViewer>
```

```csharp
// WRONG: Manually adding items to StackPanel
foreach (var item in items)
{
    ItemsPanel.Children.Add(new ItemControl { DataContext = item });
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/controls/listview-and-gridview

---

## COLLECTIONS-002: Use ItemsRepeater for Complex Layouts

**Rule**: Use `ItemsRepeater` instead of `ListView`/`GridView` when you need custom layouts, non-rectangular item arrangements, or maximum layout flexibility.

**Why**: ItemsRepeater provides lower-level virtualization control and supports custom layouts via `Layout` property. ListView/GridView have fixed layout behaviors.

**Example (Correct)**:
```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{x:Bind Photos}">
        <ItemsRepeater.Layout>
            <UniformGridLayout MinItemWidth="150"
                               MinItemHeight="150"
                               MinRowSpacing="8"
                               MinColumnSpacing="8"
                               ItemsStretch="Fill"/>
        </ItemsRepeater.Layout>
        <ItemsRepeater.ItemTemplate>
            <DataTemplate x:DataType="local:Photo">
                <Image Source="{x:Bind ThumbnailUrl}" Stretch="UniformToFill"/>
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

**ItemsRepeater vs ListView**:
| Feature | ListView | ItemsRepeater |
|---------|----------|---------------|
| Built-in selection | Yes | No (use Toolkit extensions) |
| Built-in header/footer | Yes | No |
| Custom layouts | Limited | Full control |
| Accessibility | Automatic | Manual |
| Performance overhead | Higher | Lower |

**Uno Platform Notes**: Use `ItemsRepeaterExtensions` from Uno Toolkit for selection support and incremental loading.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/controls/items-repeater

---

## COLLECTIONS-003: Avoid x:Name in DataTemplates

**Rule**: Do not use `x:Name` on elements inside `DataTemplate` for virtualized lists. Use x:Bind directly or handle `ContainerContentChanging`.

**Why**: `x:Name` in DataTemplates prevents efficient container recycling. The named elements create references that keep containers alive, defeating virtualization.

**Example (Correct)**:
```xml
<DataTemplate x:DataType="local:Item">
    <Grid Padding="12">
        <!-- No x:Name - use x:Bind directly -->
        <TextBlock Text="{x:Bind Title}"/>
        <TextBlock Text="{x:Bind Subtitle}" Grid.Row="1"/>
        <Image Source="{x:Bind ImageUrl}" Grid.Column="1"/>
    </Grid>
</DataTemplate>
```

**If you need element access, use ContainerContentChanging**:
```csharp
private void ListView_ContainerContentChanging(ListViewBase sender,
    ContainerContentChangingEventArgs args)
{
    if (args.Phase == 0)
    {
        // Find elements by walking visual tree or use Tag
        var grid = args.ItemContainer.ContentTemplateRoot as Grid;
        // Configure element...

        args.RegisterUpdateCallback(ListView_ContainerContentChanging);
    }
}
```

**Common Mistakes**:
```xml
<!-- WRONG: x:Name prevents recycling -->
<DataTemplate x:DataType="local:Item">
    <Grid>
        <TextBlock x:Name="TitleText" Text="{x:Bind Title}"/>
        <Image x:Name="ItemImage" Source="{x:Bind ImageUrl}"/>
    </Grid>
</DataTemplate>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-gridview-and-listview

---

## COLLECTIONS-004: Use ContainerContentChanging for Phased Loading

**Rule**: For complex item templates, use `ContainerContentChanging` to load content in phases - show essential content first, defer expensive content.

**Why**: Phased loading keeps scrolling smooth by showing basic content immediately and loading images/complex content after the scroll stabilizes.

**Example (Correct)**:
```xml
<ListView ItemsSource="{x:Bind Items}"
          ContainerContentChanging="ListView_ContainerContentChanging">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <Grid Height="72">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="72"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>

                <!-- Phase 0: Placeholder -->
                <Border x:Name="ImagePlaceholder" Background="LightGray"/>

                <!-- Phase 1: Image (loaded later) -->
                <Image x:Name="ItemImage" x:Phase="1"
                       Source="{x:Bind ImageUrl}" Opacity="0"/>

                <StackPanel Grid.Column="1" VerticalAlignment="Center" Margin="12,0">
                    <TextBlock Text="{x:Bind Title}"/>
                    <TextBlock x:Phase="1" Text="{x:Bind Description}"
                               Foreground="Gray"/>
                </StackPanel>
            </Grid>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

```csharp
private void ListView_ContainerContentChanging(ListViewBase sender,
    ContainerContentChangingEventArgs args)
{
    if (args.InRecycleQueue)
    {
        // Reset state when container is recycled
        var image = args.ItemContainer.ContentTemplateRoot
            .FindDescendant<Image>("ItemImage");
        if (image != null) image.Opacity = 0;
        return;
    }

    if (args.Phase == 1)
    {
        // Fade in image after it loads
        var image = args.ItemContainer.ContentTemplateRoot
            .FindDescendant<Image>("ItemImage");
        if (image != null) image.Opacity = 1;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/performance/optimize-gridview-and-listview#container-recycling

---

## COLLECTIONS-005: Optimize DataTemplateSelector

**Rule**: Keep `DataTemplateSelector.SelectTemplateCore()` as fast as possible. Use enum comparisons or dictionary lookups instead of string operations or reflection.

**Why**: SelectTemplateCore is called for every item during scrolling. Slow selection logic causes scroll jank.

**Example (Correct)**:
```csharp
public enum MessageType { Text, Image, Video, File }

public class MessageTemplateSelector : DataTemplateSelector
{
    public DataTemplate TextTemplate { get; set; }
    public DataTemplate ImageTemplate { get; set; }
    public DataTemplate VideoTemplate { get; set; }
    public DataTemplate FileTemplate { get; set; }

    protected override DataTemplate SelectTemplateCore(object item)
    {
        if (item is Message message)
        {
            return message.Type switch
            {
                MessageType.Text => TextTemplate,
                MessageType.Image => ImageTemplate,
                MessageType.Video => VideoTemplate,
                MessageType.File => FileTemplate,
                _ => TextTemplate
            };
        }
        return TextTemplate;
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: String comparison is slow
protected override DataTemplate SelectTemplateCore(object item)
{
    var message = item as Message;
    if (message?.TypeString?.ToLower() == "text") // String allocation + comparison
        return TextTemplate;
    // ...
}

// WRONG: Reflection is very slow
protected override DataTemplate SelectTemplateCore(object item)
{
    var typeName = item.GetType().Name; // Reflection
    // ...
}
```

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.datatemplateselector

---

## COLLECTIONS-006: Set ItemsPanel for Optimal Virtualization

**Rule**: Use `ItemsStackPanel` (default) for vertical lists, `ItemsWrapGrid` for grids. Avoid `StackPanel` or `WrapPanel` as ItemsPanel.

**Why**: `ItemsStackPanel` and `ItemsWrapGrid` are virtualized. `StackPanel` and `WrapPanel` create all items immediately.

**Example (Correct)**:
```xml
<!-- Vertical list (default, but explicit) -->
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemsPanel>
        <ItemsPanelTemplate>
            <ItemsStackPanel Orientation="Vertical"/>
        </ItemsPanelTemplate>
    </ListView.ItemsPanel>
</ListView>

<!-- Horizontal list -->
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemsPanel>
        <ItemsPanelTemplate>
            <ItemsStackPanel Orientation="Horizontal"/>
        </ItemsPanelTemplate>
    </ListView.ItemsPanel>
</ListView>

<!-- Grid layout -->
<GridView ItemsSource="{x:Bind Items}">
    <GridView.ItemsPanel>
        <ItemsPanelTemplate>
            <ItemsWrapGrid Orientation="Horizontal"
                           MaximumRowsOrColumns="4"/>
        </ItemsPanelTemplate>
    </GridView.ItemsPanel>
</GridView>
```

**Common Mistakes**:
```xml
<!-- WRONG: StackPanel defeats virtualization -->
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemsPanel>
        <ItemsPanelTemplate>
            <StackPanel/>  <!-- No virtualization! -->
        </ItemsPanelTemplate>
    </ListView.ItemsPanel>
</ListView>
```

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.itemsstackpanel

---

## COLLECTIONS-007: Use ObservableCollection Properly

**Rule**: Use `ObservableCollection<T>` for bound collections that change. For bulk updates, use `AddRange` extensions or replace the entire collection.

**Why**: `ObservableCollection` fires `CollectionChanged` on every Add/Remove. Adding 1000 items fires 1000 events. Use bulk operations or collection replacement for large updates.

**Example (Correct)**:
```csharp
// For bulk additions - replace collection or use batch helper
public ObservableCollection<Item> Items { get; } = new();

public async Task RefreshItemsAsync()
{
    var newItems = await _service.GetItemsAsync();

    // Option 1: Clear and add (better for binding)
    Items.Clear();
    foreach (var item in newItems)
    {
        Items.Add(item);
    }

    // Option 2: Replace collection (requires re-binding or PropertyChanged)
    // Items = new ObservableCollection<Item>(newItems);
    // OnPropertyChanged(nameof(Items));
}

// For incremental loading - individual adds are fine
public void AddItem(Item item)
{
    Items.Add(item);
}
```

**With CommunityToolkit**:
```csharp
using CommunityToolkit.Mvvm.ComponentModel;

// ObservableRangeCollection supports bulk operations
public ObservableRangeCollection<Item> Items { get; } = new();

public async Task RefreshItemsAsync()
{
    var newItems = await _service.GetItemsAsync();
    Items.ReplaceRange(newItems); // Single notification
}
```

**Common Mistakes**:
```csharp
// WRONG: 1000 individual notifications
foreach (var item in await _service.GetItemsAsync())
{
    Items.Add(item); // Fires CollectionChanged each time
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/api/system.collections.objectmodel.observablecollection-1
