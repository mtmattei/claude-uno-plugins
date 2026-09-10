# Data Binding Extensions

## Impact: HIGH
AncestorBinding, ItemsControlBinding, and ProgressExtensions solve common binding limitations in WinUI/Uno Platform. Forgetting the `DataContext.` prefix in Path, using ElementName inside DataTemplates, or binding ProgressExtensions.IsActive to a non-ILoadable source causes silent binding failures.

---

## Prerequisites

```xml
xmlns:utu="using:Uno.Toolkit.UI"
```

```xml
<UnoFeatures>Toolkit;Material;</UnoFeatures>
```

---

## AncestorBinding

**Impact: CRITICAL**

WinUI has no built-in `RelativeSource FindAncestor`. AncestorBinding walks the visual tree to find an ancestor of the specified type and binds to its property. Essential for accessing parent ViewModel properties from inside DataTemplates.

### Syntax

```xml
{utu:AncestorBinding AncestorType=TypeName, Path=PropertyPath}
```

- `AncestorType`: The type to search for up the visual tree (e.g., `Page`, `ListView`, `Grid`)
- `Path`: The property path on the found ancestor

### CORRECT
```xml
<!-- Access Page's DataContext (parent ViewModel) from inside a DataTemplate -->
<Page x:Name="MyPage">
    <ListView ItemsSource="{Binding Items}">
        <ListView.ItemTemplate>
            <DataTemplate>
                <utu:AutoLayout Orientation="Horizontal" Spacing="8" Padding="12">
                    <!-- Normal item binding -->
                    <TextBlock Text="{Binding Name}"
                               Style="{StaticResource BodyLarge}" />

                    <!-- Bind to parent ViewModel's command -->
                    <Button Content="Delete"
                            Command="{utu:AncestorBinding AncestorType=Page,
                                                           Path=DataContext.DeleteItemCommand}"
                            CommandParameter="{Binding}" />

                    <!-- Bind to parent ViewModel's property -->
                    <TextBlock Text="{utu:AncestorBinding AncestorType=Page,
                                                           Path=DataContext.CategoryLabel}"
                               Style="{StaticResource LabelSmall}" />
                </utu:AutoLayout>
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</Page>

<!-- Access ancestor element's own property (not DataContext) -->
<Grid Tag="SomeValue">
    <ListView ItemsSource="{Binding Items}">
        <ListView.ItemTemplate>
            <DataTemplate>
                <!-- Path=Tag accesses the Grid's Tag property directly -->
                <TextBlock Text="{utu:AncestorBinding AncestorType=Grid, Path=Tag}" />
            </DataTemplate>
        </ListView.ItemTemplate>
    </ListView>
</Grid>
```

### WRONG
```xml
<!-- WRONG: Missing DataContext. prefix when accessing ViewModel properties -->
<Button Command="{utu:AncestorBinding AncestorType=Page,
                                       Path=DeleteItemCommand}" />
<!-- This looks for Page.DeleteItemCommand (a property on the Page element itself) -->
<!-- FIX: Path=DataContext.DeleteItemCommand -->

<!-- WRONG: Using ElementName binding inside a DataTemplate -->
<Button Command="{Binding DataContext.DeleteCommand, ElementName=MyPage}" />
<!-- ElementName may not resolve correctly inside DataTemplates across all platforms -->
<!-- FIX: Use AncestorBinding instead -->

<!-- WRONG: Specifying AncestorType as the template host when you need the page -->
<Button Command="{utu:AncestorBinding AncestorType=ListView,
                                       Path=DataContext.DeleteCommand}" />
<!-- This finds the ListView, which may share the Page's DataContext but is fragile -->
<!-- FIX: Walk up to Page for the most reliable ancestor type -->
```

---

## ItemsControlBinding

**Impact: HIGH**

A shortcut for `AncestorBinding` that automatically targets the nearest parent `ItemsControl` (ListView, GridView, ItemsRepeater, etc.). Eliminates the need to specify `AncestorType` when binding to the items control's DataContext.

### Syntax

```xml
{utu:ItemsControlBinding Path=PropertyPath}
```

### CORRECT
```xml
<!-- Access the ListView's DataContext (parent ViewModel) -->
<ListView ItemsSource="{Binding Items}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <utu:AutoLayout Orientation="Horizontal" Spacing="8" Padding="12">
                <TextBlock Text="{Binding Name}"
                           Style="{StaticResource BodyLarge}" />

                <!-- Command on parent ViewModel via ItemsControlBinding -->
                <Button Content="Edit"
                        Command="{utu:ItemsControlBinding Path=DataContext.EditItemCommand}"
                        CommandParameter="{Binding}" />

                <!-- Property on parent ViewModel -->
                <TextBlock Text="{utu:ItemsControlBinding Path=DataContext.StatusMessage}"
                           Style="{StaticResource LabelSmall}"
                           Foreground="{ThemeResource OnSurfaceVariantBrush}" />
            </utu:AutoLayout>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>

<!-- Access ItemsControl's own property (e.g., Tag) -->
<ListView ItemsSource="{Binding Items}" Tag="Shared Value">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{utu:ItemsControlBinding Path=Tag}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

### WRONG
```xml
<!-- WRONG: Missing DataContext. prefix -->
<Button Command="{utu:ItemsControlBinding Path=EditItemCommand}" />
<!-- This looks for a property named EditItemCommand on the ListView itself -->
<!-- FIX: Path=DataContext.EditItemCommand -->

<!-- WRONG: Using ItemsControlBinding when the nearest ItemsControl is not the right one -->
<!-- In nested ItemsControls, ItemsControlBinding finds the NEAREST parent -->
<ListView ItemsSource="{Binding Categories}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <ListView ItemsSource="{Binding Items}">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <!-- This binds to the INNER ListView's DataContext, not the outer one -->
                        <Button Command="{utu:ItemsControlBinding Path=DataContext.Command}" />
                        <!-- FIX: Use AncestorBinding with AncestorType=Page for the outer context -->
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

---

## AncestorBinding vs ItemsControlBinding: When to Use Which

**Impact: MEDIUM**

| Scenario | Use |
|----------|-----|
| Inside a single-level ItemsControl DataTemplate | `ItemsControlBinding` (simpler) |
| Inside nested ItemsControls needing outer context | `AncestorBinding` with `AncestorType=Page` |
| Need to access a non-ItemsControl ancestor | `AncestorBinding` with specific type |
| Need to access the Page's DataContext | `AncestorBinding AncestorType=Page` |

### CORRECT
```xml
<!-- Simple case: ItemsControlBinding is cleaner -->
<ListView ItemsSource="{Binding Items}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <Button Command="{utu:ItemsControlBinding Path=DataContext.DeleteCommand}"
                    CommandParameter="{Binding}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>

<!-- Nested case: AncestorBinding escapes to the correct level -->
<ListView ItemsSource="{Binding Groups}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <ItemsRepeater ItemsSource="{Binding Children}">
                <ItemsRepeater.ItemTemplate>
                    <DataTemplate>
                        <Button Command="{utu:AncestorBinding AncestorType=Page,
                                                               Path=DataContext.GlobalCommand}"
                                CommandParameter="{Binding}" />
                    </DataTemplate>
                </ItemsRepeater.ItemTemplate>
            </ItemsRepeater>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

---

## ProgressExtensions

**Impact: MEDIUM**

`ProgressExtensions.IsActive` recursively activates nested `ProgressBar` and `ProgressRing` controls. Apply it to a container and all progress indicators within become active/inactive together.

### CORRECT
```xml
<!-- Activate all nested progress indicators at once -->
<Grid utu:ProgressExtensions.IsActive="{Binding IsLoading}">
    <ProgressBar />
    <StackPanel>
        <ProgressRing Width="24" Height="24" />
        <TextBlock Text="Loading..." Style="{StaticResource BodyMedium}" />
    </StackPanel>
</Grid>

<!-- Combine with LoadingView for comprehensive loading UX -->
<utu:LoadingView Source="{Binding LoadCommand}">
    <utu:LoadingView.LoadingContent>
        <Grid utu:ProgressExtensions.IsActive="True">
            <ProgressRing Width="32" Height="32" />
        </Grid>
    </utu:LoadingView.LoadingContent>
    <ListView ItemsSource="{Binding Results}" />
</utu:LoadingView>
```

### WRONG
```xml
<!-- WRONG: Setting IsActive on each ProgressRing individually -->
<ProgressRing IsActive="{Binding IsLoading}" />
<ProgressBar IsIndeterminate="{Binding IsLoading}" />
<!-- This works but is repetitive. FIX: Use ProgressExtensions.IsActive on the parent -->

<!-- WRONG: Expecting ProgressExtensions to control visibility -->
<!-- IsActive only controls the progress animation, not Visibility -->
<Grid utu:ProgressExtensions.IsActive="{Binding IsLoading}">
    <ProgressRing />
</Grid>
<!-- The ProgressRing is still visible (just not animating) when IsActive is false -->
<!-- FIX: Also bind Visibility if you want to hide the indicator -->
```

---

## Platform Notes

- `AncestorBinding` and `ItemsControlBinding` are available on all WinUI 3 platforms and all non-Windows UWP platforms. They are **not** available on Windows UWP.
- Both markup extensions walk the visual tree at binding time. Very deep trees with many ancestors may have a minor performance impact.
- `ProgressExtensions.IsActive` works on all platforms uniformly.

---

## References

- [AncestorBinding and ItemsControlBinding](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/ancestor-itemscontrol-binding.html)
- [ProgressExtensions](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/progress-extensions.html)
