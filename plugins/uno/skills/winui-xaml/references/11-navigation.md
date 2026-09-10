# Navigation Skills - WinUI 3 XAML

## NAV-001: Use Frame.Navigate() Correctly

**Rule**: Pass lightweight parameters to `Frame.Navigate()`. For complex objects, use navigation services, messaging, or shared state.

**Why**: Navigation parameters are serialized for suspension. Large objects cause suspension failures and memory issues. Pass IDs or keys instead of full objects.

**Example (Correct)**:
```csharp
// Pass ID, not full object
private void ListView_ItemClick(object sender, ItemClickEventArgs e)
{
    var item = e.ClickedItem as Product;
    Frame.Navigate(typeof(ProductDetailPage), item.Id);  // Pass ID
}

// Destination page retrieves full object
protected override async void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);

    if (e.Parameter is int productId)
    {
        ViewModel.Product = await _productService.GetProductAsync(productId);
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Passing large complex object
Frame.Navigate(typeof(DetailPage), largeDataObject);  // Serialization issues

// WRONG: Passing non-serializable object
Frame.Navigate(typeof(DetailPage), myViewModel);  // Fails on suspension
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/basics/navigation-history-and-backwards-navigation

---

## NAV-002: Handle NavigationFailed Event

**Rule**: Subscribe to `Frame.NavigationFailed` to catch and handle navigation errors gracefully.

**Why**: Silent navigation failures leave users stuck. Proper error handling provides feedback and recovery options.

**Example (Correct)**:
```csharp
// In App.xaml.cs or root frame setup
rootFrame.NavigationFailed += OnNavigationFailed;

private void OnNavigationFailed(object sender, NavigationFailedEventArgs e)
{
    // Log the error
    Debug.WriteLine($"Navigation failed: {e.SourcePageType?.Name} - {e.Exception}");

    // Show error to user (don't crash)
    var dialog = new ContentDialog
    {
        Title = "Navigation Error",
        Content = "Unable to load the requested page. Please try again.",
        CloseButtonText = "OK"
    };

    _ = dialog.ShowAsync();

    e.Handled = true;  // Prevent app crash
}
```

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.frame.navigationfailed

---

## NAV-003: Implement Page Caching Appropriately

**Rule**: Use `NavigationCacheMode` to cache pages that are expensive to create or should preserve state. Be mindful of memory.

**Why**: Page caching avoids recreation cost on back navigation. But cached pages consume memory. Balance performance vs memory.

**Example (Correct)**:
```csharp
public sealed partial class SearchResultsPage : Page
{
    public SearchResultsPage()
    {
        InitializeComponent();

        // Cache this page - preserves search results and scroll position
        NavigationCacheMode = NavigationCacheMode.Enabled;
    }

    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        base.OnNavigatedTo(e);

        // Check if this is a fresh navigation or returning to cached page
        if (e.NavigationMode == NavigationMode.New)
        {
            // Fresh navigation - load new data
            ViewModel.LoadResults(e.Parameter as string);
        }
        // else: returning to cached page, data already loaded
    }
}
```

**NavigationCacheMode Options**:
| Mode | Behavior | Use Case |
|------|----------|----------|
| `Disabled` | Always recreate page | Simple pages, forms |
| `Enabled` | Cache during session | Lists, search results |
| `Required` | Always cache (survives Frame recreation) | Critical state pages |

**Common Mistakes**:
```csharp
// WRONG: Caching all pages - memory bloat
public BasePage()
{
    NavigationCacheMode = NavigationCacheMode.Required;  // Every page cached!
}
```

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.page.navigationcachemode

---

## NAV-004: Handle Back Navigation

**Rule**: Always handle back navigation. Check for unsaved changes. Register for system back button on supported platforms.

**Why**: Users expect back button to work consistently. Unsaved changes should prompt confirmation. System back integration improves UX.

**Example (Correct)**:
```csharp
public sealed partial class EditPage : Page
{
    private bool _hasUnsavedChanges;

    public EditPage()
    {
        InitializeComponent();

        // Register for system back button
        SystemNavigationManager.GetForCurrentView().BackRequested += OnBackRequested;
    }

    private async void OnBackRequested(object sender, BackRequestedEventArgs e)
    {
        if (_hasUnsavedChanges)
        {
            e.Handled = true;  // Cancel automatic back navigation

            var dialog = new ContentDialog
            {
                Title = "Unsaved changes",
                Content = "Do you want to discard your changes?",
                PrimaryButtonText = "Discard",
                CloseButtonText = "Cancel"
            };

            if (await dialog.ShowAsync() == ContentDialogResult.Primary)
            {
                _hasUnsavedChanges = false;
                if (Frame.CanGoBack)
                    Frame.GoBack();
            }
        }
    }

    protected override void OnNavigatedFrom(NavigationEventArgs e)
    {
        base.OnNavigatedFrom(e);
        SystemNavigationManager.GetForCurrentView().BackRequested -= OnBackRequested;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/basics/navigation-history-and-backwards-navigation

---

## NAV-005: Use NavigationView for App Navigation

**Rule**: Use `NavigationView` for app-level navigation with hamburger menu, back button, and settings access built-in.

**Why**: NavigationView provides standard Windows navigation patterns, accessibility support, and responsive layout (hamburger on narrow, sidebar on wide).

**Example (Correct)**:
```xml
<NavigationView x:Name="NavView"
                IsBackButtonVisible="Auto"
                BackRequested="NavView_BackRequested"
                ItemInvoked="NavView_ItemInvoked">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" Icon="Home" Tag="HomePage"/>
        <NavigationViewItem Content="Products" Icon="Shop" Tag="ProductsPage"/>
        <NavigationViewItem Content="Orders" Icon="List" Tag="OrdersPage"/>
    </NavigationView.MenuItems>

    <NavigationView.FooterMenuItems>
        <NavigationViewItem Content="Settings" Icon="Setting" Tag="SettingsPage"/>
    </NavigationView.FooterMenuItems>

    <Frame x:Name="ContentFrame"/>
</NavigationView>
```

```csharp
private void NavView_ItemInvoked(NavigationView sender, NavigationViewItemInvokedEventArgs args)
{
    if (args.InvokedItemContainer is NavigationViewItem item)
    {
        var pageTag = item.Tag as string;
        var pageType = pageTag switch
        {
            "HomePage" => typeof(HomePage),
            "ProductsPage" => typeof(ProductsPage),
            "OrdersPage" => typeof(OrdersPage),
            "SettingsPage" => typeof(SettingsPage),
            _ => null
        };

        if (pageType != null && ContentFrame.CurrentSourcePageType != pageType)
        {
            ContentFrame.Navigate(pageType);
        }
    }
}

private void NavView_BackRequested(NavigationView sender, NavigationViewBackRequestedEventArgs args)
{
    if (ContentFrame.CanGoBack)
    {
        ContentFrame.GoBack();
    }
}
```

**Uno Platform Notes**: NavigationView is fully supported. For Uno.Extensions navigation, use Region.Name and Navigation.Request attached properties.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/controls/navigationview

---

## NAV-006: Clean Up Resources on Navigation

**Rule**: In `OnNavigatedFrom`, unsubscribe events, stop timers, cancel pending operations, and release resources.

**Why**: Pages may be cached or destroyed after navigation. Failing to clean up causes memory leaks and unexpected behavior from stale event handlers.

**Example (Correct)**:
```csharp
public sealed partial class LiveDataPage : Page
{
    private readonly DispatcherQueueTimer _refreshTimer;
    private CancellationTokenSource _loadCts;

    public LiveDataPage()
    {
        InitializeComponent();

        _refreshTimer = DispatcherQueue.GetForCurrentThread().CreateTimer();
        _refreshTimer.Tick += RefreshTimer_Tick;
    }

    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        base.OnNavigatedTo(e);

        _loadCts = new CancellationTokenSource();
        _refreshTimer.Start();

        // Subscribe to events
        App.DataService.DataUpdated += OnDataUpdated;
    }

    protected override void OnNavigatedFrom(NavigationEventArgs e)
    {
        base.OnNavigatedFrom(e);

        // Stop timer
        _refreshTimer.Stop();

        // Cancel pending operations
        _loadCts?.Cancel();
        _loadCts?.Dispose();
        _loadCts = null;

        // Unsubscribe events
        App.DataService.DataUpdated -= OnDataUpdated;
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/launch-resume/app-lifecycle

---

## NAV-007: Use Deep Links and URI Activation

**Rule**: Support URI activation for deep linking into specific pages/content within your app.

**Why**: Deep links enable sharing content, notification actions, and integration with other apps. Users expect links to open relevant content directly.

**Example (Correct)**:
```csharp
// In App.xaml.cs
protected override void OnActivated(IActivatedEventArgs e)
{
    if (e is ProtocolActivatedEventArgs protocolArgs)
    {
        // URI: myapp://product/123
        var uri = protocolArgs.Uri;

        if (uri.Host == "product" && int.TryParse(uri.LocalPath.TrimStart('/'), out var productId))
        {
            EnsureWindowCreated();
            var rootFrame = Window.Current.Content as Frame;
            rootFrame.Navigate(typeof(ProductDetailPage), productId);
        }
    }

    base.OnActivated(e);
}
```

**Package.appxmanifest**:
```xml
<Extensions>
    <uap:Extension Category="windows.protocol">
        <uap:Protocol Name="myapp">
            <uap:DisplayName>My App</uap:DisplayName>
        </uap:Protocol>
    </uap:Extension>
</Extensions>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/deep-links-and-app-to-app-activation
