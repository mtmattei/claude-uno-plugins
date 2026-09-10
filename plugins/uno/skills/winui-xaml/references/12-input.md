# Input & Interaction Skills - WinUI 3 XAML

## INPUT-001: Set Minimum Touch Target Size

**Rule**: Ensure all interactive elements have minimum 44x44 pixel touch targets (48x48 dp recommended).

**Why**: Small touch targets frustrate users and are inaccessible to users with motor impairments. WCAG 2.1 requires adequate target sizing.

**Example (Correct)**:
```xml
<!-- Button with adequate touch target -->
<Button Content="Submit" MinWidth="88" MinHeight="44"/>

<!-- Icon button with explicit size -->
<Button MinWidth="44" MinHeight="44" Padding="10"
        AutomationProperties.Name="Delete">
    <FontIcon Glyph="&#xE74D;" FontSize="16"/>
</Button>

<!-- Small visual, large touch target -->
<Button Width="44" Height="44" Background="Transparent"
        AutomationProperties.Name="Status indicator">
    <Ellipse Width="12" Height="12" Fill="{ThemeResource SystemFillColorSuccessBrush}"/>
</Button>

<!-- List items with adequate height -->
<ListView.ItemTemplate>
    <DataTemplate>
        <Grid MinHeight="48" Padding="16,12">
            <TextBlock Text="{x:Bind Name}"/>
        </Grid>
    </DataTemplate>
</ListView.ItemTemplate>
```

**Common Mistakes**:
```xml
<!-- WRONG: Too small for touch -->
<Button Width="20" Height="20" Content="X"/>

<!-- WRONG: Icon without padding -->
<Button>
    <FontIcon Glyph="&#xE711;"/>  <!-- Glyph is small, no padding -->
</Button>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/guidelines-for-targeting

---

## INPUT-002: Set InputScope for Text Input

**Rule**: Always set `InputScope` on TextBox controls to display the appropriate virtual keyboard on touch devices.

**Why**: Correct InputScope shows relevant keyboard layout (numeric, email, URL, phone), improving input efficiency and reducing errors.

**Example (Correct)**:
```xml
<!-- Email input -->
<TextBox Header="Email"
         InputScope="EmailSmtpAddress"
         PlaceholderText="name@example.com"/>

<!-- Phone number -->
<TextBox Header="Phone"
         InputScope="TelephoneNumber"
         PlaceholderText="+1 (555) 123-4567"/>

<!-- URL input -->
<TextBox Header="Website"
         InputScope="Url"
         PlaceholderText="https://example.com"/>

<!-- Numeric input -->
<TextBox Header="Quantity"
         InputScope="Number"/>

<!-- Password (use PasswordBox instead) -->
<PasswordBox Header="Password"
             PlaceholderText="Enter password"/>

<!-- Search -->
<TextBox Header="Search"
         InputScope="Search"
         PlaceholderText="Search products..."/>
```

**Common InputScope Values**:
| InputScope | Keyboard |
|------------|----------|
| `Default` | Standard keyboard |
| `EmailSmtpAddress` | Email keyboard with @ |
| `TelephoneNumber` | Numeric dial pad |
| `Number` | Numeric keyboard |
| `Url` | URL keyboard with .com |
| `Search` | Search keyboard with search key |

**Common Mistakes**:
```xml
<!-- WRONG: No InputScope for email -->
<TextBox Header="Email"/>  <!-- Shows standard keyboard -->
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/use-input-scope-to-change-the-touch-keyboard

---

## INPUT-003: Handle Pointer Events for Cross-Input Support

**Rule**: Use Pointer events (PointerPressed, PointerReleased, PointerMoved) for cross-input compatibility instead of separate mouse/touch/pen handlers.

**Why**: Pointer events work with mouse, touch, and pen uniformly. Avoids writing separate handlers for each input type.

**Example (Correct)**:
```csharp
public sealed partial class DrawingCanvas : UserControl
{
    public DrawingCanvas()
    {
        InitializeComponent();

        // Single set of handlers for all input types
        PointerPressed += OnPointerPressed;
        PointerMoved += OnPointerMoved;
        PointerReleased += OnPointerReleased;
        PointerCanceled += OnPointerCanceled;
    }

    private void OnPointerPressed(object sender, PointerRoutedEventArgs e)
    {
        var point = e.GetCurrentPoint(this);

        // Capture for drag operations
        CapturePointer(e.Pointer);

        // Start drawing
        _currentPath = new Path();
        _isDrawing = true;

        e.Handled = true;
    }

    private void OnPointerMoved(object sender, PointerRoutedEventArgs e)
    {
        if (!_isDrawing) return;

        var point = e.GetCurrentPoint(this);

        // Check pressure for pen input
        if (point.Properties.Pressure > 0)
        {
            // Vary stroke width based on pressure
            var strokeWidth = point.Properties.Pressure * 10;
        }

        // Add point to path
        e.Handled = true;
    }

    private void OnPointerReleased(object sender, PointerRoutedEventArgs e)
    {
        ReleasePointerCapture(e.Pointer);
        _isDrawing = false;
        e.Handled = true;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/handle-pointer-input

---

## INPUT-004: Implement Keyboard Shortcuts

**Rule**: Provide keyboard shortcuts for common actions. Use `KeyboardAccelerator` for discoverable shortcuts shown in menus.

**Why**: Keyboard users rely on shortcuts for efficiency. KeyboardAccelerator integrates with menus to show shortcut hints.

**Example (Correct)**:
```xml
<!-- Menu with keyboard accelerators -->
<MenuBar>
    <MenuBarItem Title="File">
        <MenuFlyoutItem Text="New" Icon="Add">
            <MenuFlyoutItem.KeyboardAccelerators>
                <KeyboardAccelerator Key="N" Modifiers="Control"/>
            </MenuFlyoutItem.KeyboardAccelerators>
        </MenuFlyoutItem>

        <MenuFlyoutItem Text="Save" Icon="Save" Command="{x:Bind ViewModel.SaveCommand}">
            <MenuFlyoutItem.KeyboardAccelerators>
                <KeyboardAccelerator Key="S" Modifiers="Control"/>
            </MenuFlyoutItem.KeyboardAccelerators>
        </MenuFlyoutItem>
    </MenuBarItem>
</MenuBar>

<!-- Page-level shortcuts -->
<Page.KeyboardAccelerators>
    <KeyboardAccelerator Key="F" Modifiers="Control"
                         Invoked="SearchAccelerator_Invoked"/>
    <KeyboardAccelerator Key="Escape"
                         Invoked="EscapeAccelerator_Invoked"/>
</Page.KeyboardAccelerators>
```

```csharp
private void SearchAccelerator_Invoked(KeyboardAccelerator sender,
    KeyboardAcceleratorInvokedEventArgs args)
{
    SearchBox.Focus(FocusState.Keyboard);
    args.Handled = true;
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/keyboard-accelerators

---

## INPUT-005: Handle Focus Correctly

**Rule**: Manage focus programmatically for accessibility and UX. Set initial focus, trap focus in dialogs, restore focus after actions.

**Why**: Proper focus management is essential for keyboard navigation and screen reader users. Focus should follow logical user expectations.

**Example (Correct)**:
```csharp
// Set initial focus when page loads
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);

    // Focus first interactive element
    _ = Dispatcher.TryRunAsync(CoreDispatcherPriority.Normal, () =>
    {
        FirstNameTextBox.Focus(FocusState.Programmatic);
    });
}

// Restore focus after dialog
private async void ShowFilterDialog()
{
    var focusedElement = FocusManager.GetFocusedElement() as Control;

    var dialog = new FilterDialog();
    await dialog.ShowAsync();

    // Restore focus
    focusedElement?.Focus(FocusState.Programmatic);
}

// Focus trap in custom dialog/overlay
public sealed partial class CustomOverlay : UserControl
{
    public CustomOverlay()
    {
        InitializeComponent();

        // Trap Tab key within overlay
        PreviewKeyDown += OnPreviewKeyDown;
    }

    private void OnPreviewKeyDown(object sender, KeyRoutedEventArgs e)
    {
        if (e.Key == VirtualKey.Tab)
        {
            var focusedElement = FocusManager.GetFocusedElement();
            var lastElement = CloseButton;
            var firstElement = ContentArea;

            if (focusedElement == lastElement && !IsShiftPressed())
            {
                firstElement.Focus(FocusState.Keyboard);
                e.Handled = true;
            }
        }
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/focus-navigation

---

## INPUT-006: Support Drag and Drop

**Rule**: Implement drag-and-drop using `AllowDrop`, `CanDrag`, and the DragStarting/Drop events for intuitive data transfer.

**Why**: Drag-and-drop is an intuitive interaction pattern users expect. It works across mouse, touch, and pen input.

**Example (Correct)**:
```xml
<!-- Draggable item -->
<Grid CanDrag="True"
      DragStarting="Item_DragStarting">
    <TextBlock Text="{x:Bind Name}"/>
</Grid>

<!-- Drop target -->
<ListView AllowDrop="True"
          DragOver="ListView_DragOver"
          Drop="ListView_Drop">
    <!-- Items -->
</ListView>
```

```csharp
private void Item_DragStarting(UIElement sender, DragStartingEventArgs args)
{
    var item = (sender as FrameworkElement)?.DataContext as MyItem;

    args.Data.SetText(item.Id.ToString());
    args.Data.RequestedOperation = DataPackageOperation.Move;
}

private void ListView_DragOver(object sender, DragEventArgs e)
{
    if (e.DataView.Contains(StandardDataFormats.Text))
    {
        e.AcceptedOperation = DataPackageOperation.Move;
        e.DragUIOverride.Caption = "Move here";
    }
}

private async void ListView_Drop(object sender, DragEventArgs e)
{
    if (e.DataView.Contains(StandardDataFormats.Text))
    {
        var itemId = await e.DataView.GetTextAsync();
        // Handle the dropped item
        ViewModel.MoveItem(int.Parse(itemId), GetDropIndex(e));
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/drag-and-drop

---

## INPUT-007: Debounce Rapid Input

**Rule**: Debounce text input for search/filter operations. Throttle rapid button clicks to prevent duplicate actions.

**Why**: Processing every keystroke wastes resources and causes flickering. Rapid clicks can trigger duplicate API calls or navigation.

**Example (Correct)**:
```csharp
// Debounced search
private DispatcherQueueTimer _searchDebounceTimer;

private void SearchBox_TextChanged(object sender, TextChangedEventArgs e)
{
    _searchDebounceTimer?.Stop();

    _searchDebounceTimer = DispatcherQueue.GetForCurrentThread().CreateTimer();
    _searchDebounceTimer.Interval = TimeSpan.FromMilliseconds(300);
    _searchDebounceTimer.IsRepeating = false;
    _searchDebounceTimer.Tick += (s, args) =>
    {
        ViewModel.SearchCommand.Execute(SearchBox.Text);
    };
    _searchDebounceTimer.Start();
}

// Throttled button click
private DateTime _lastClickTime;
private readonly TimeSpan _clickThrottle = TimeSpan.FromMilliseconds(500);

private async void SubmitButton_Click(object sender, RoutedEventArgs e)
{
    if (DateTime.Now - _lastClickTime < _clickThrottle)
        return;

    _lastClickTime = DateTime.Now;

    SubmitButton.IsEnabled = false;
    try
    {
        await ViewModel.SubmitAsync();
    }
    finally
    {
        SubmitButton.IsEnabled = true;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/keyboard-interactions
