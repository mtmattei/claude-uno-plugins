# Async & Threading Skills - WinUI 3 XAML

## ASYNC-001: Never Block the UI Thread

**Rule**: Never use `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()` on tasks from the UI thread. Always use `async`/`await`.

**Why**: Blocking the UI thread causes app freezes, unresponsive UI, and potential deadlocks. Users perceive any delay > 100ms as lag. File I/O, network calls, and database operations can take seconds.

**Example (Correct)**:
```csharp
public async Task LoadDataAsync()
{
    // Async all the way
    var data = await _service.GetDataAsync();

    // Already on UI thread after await with default context
    DataList.ItemsSource = data;
}

private async void LoadButton_Click(object sender, RoutedEventArgs e)
{
    LoadingIndicator.IsActive = true;
    try
    {
        await LoadDataAsync();
    }
    finally
    {
        LoadingIndicator.IsActive = false;
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Blocks UI thread - causes freeze
public void LoadData()
{
    var data = _service.GetDataAsync().Result; // DEADLOCK RISK!
    DataList.ItemsSource = data;
}

// WRONG: GetAwaiter().GetResult() also blocks
public void SaveFile(string content)
{
    var file = StorageFile.GetFileFromPathAsync(path).GetAwaiter().GetResult();
    FileIO.WriteTextAsync(file, content).GetAwaiter().GetResult();
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/keep-the-ui-thread-responsive

---

## ASYNC-002: Use Task.WhenAll for Independent Operations

**Rule**: When multiple async operations have no dependencies, execute them concurrently with `Task.WhenAll()`.

**Why**: Sequential awaits add latency. If operations A (300ms), B (200ms), C (150ms) are independent, sequential = 650ms, parallel = 300ms (max of all).

**Example (Correct)**:
```csharp
public async Task<DashboardData> LoadDashboardAsync()
{
    // Start all tasks
    var userTask = _userService.GetUserAsync();
    var postsTask = _postService.GetRecentPostsAsync();
    var statsTask = _statsService.GetStatsAsync();

    // Wait for all to complete
    await Task.WhenAll(userTask, postsTask, statsTask);

    return new DashboardData
    {
        User = userTask.Result,      // Already completed, safe to access
        Posts = postsTask.Result,
        Stats = statsTask.Result
    };
}
```

**Common Mistakes**:
```csharp
// WRONG: Sequential execution wastes time
public async Task<DashboardData> LoadDashboardAsync()
{
    var user = await _userService.GetUserAsync();    // Wait 300ms
    var posts = await _postService.GetRecentPostsAsync(); // Then wait 200ms
    var stats = await _statsService.GetStatsAsync();      // Then wait 150ms
    // Total: 650ms instead of 300ms

    return new DashboardData { User = user, Posts = posts, Stats = stats };
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall

---

## ASYNC-003: Sequence Dependent Operations Only

**Rule**: Use sequential await only when operations depend on previous results. Parallelize everything else.

**Why**: Identify the dependency graph. Operations depending on the same input can run in parallel even if they follow a dependent operation.

**Example (Correct)**:
```csharp
public async Task<UserDashboard> LoadUserDashboardAsync(int userId)
{
    // Step 1: Get user (required for subsequent calls)
    var user = await _userService.GetUserAsync(userId);

    // Step 2: Parallelize operations that depend only on user
    var ordersTask = _orderService.GetOrdersAsync(user.Id);
    var reviewsTask = _reviewService.GetReviewsAsync(user.Id);
    var recommendationsTask = _recommendationService.GetRecommendationsAsync(user.PreferenceProfile);

    await Task.WhenAll(ordersTask, reviewsTask, recommendationsTask);

    return new UserDashboard
    {
        User = user,
        Orders = ordersTask.Result,
        Reviews = reviewsTask.Result,
        Recommendations = recommendationsTask.Result
    };
}
```

**Common Mistakes**:
```csharp
// WRONG: All sequential despite only needing userId
var user = await _userService.GetUserAsync(userId);
var orders = await _orderService.GetOrdersAsync(user.Id);
var reviews = await _reviewService.GetReviewsAsync(user.Id);
var recommendations = await _recommendationService.GetRecommendationsAsync(user.PreferenceProfile);
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/

---

## ASYNC-004: Use DispatcherQueue for Background Thread Updates

**Rule**: Use `DispatcherQueue.TryEnqueue()` to marshal UI updates from background threads. In WinUI 3, use `DispatcherQueue` (not `CoreDispatcher`).

**Why**: UI elements can only be accessed from the UI thread. Background thread access throws exceptions. DispatcherQueue is the WinUI 3 replacement for CoreDispatcher.

**Example (Correct)**:
```csharp
public sealed partial class MainPage : Page
{
    private readonly DispatcherQueue _dispatcherQueue;

    public MainPage()
    {
        InitializeComponent();
        _dispatcherQueue = DispatcherQueue.GetForCurrentThread();
    }

    private async void StartBackgroundWork()
    {
        await Task.Run(() =>
        {
            // Heavy computation on background thread
            var result = ProcessData();

            // Marshal back to UI thread
            _dispatcherQueue.TryEnqueue(() =>
            {
                ResultText.Text = result;
                ProgressBar.Value = 100;
            });
        });
    }
}
```

**For event handlers from non-UI sources**:
```csharp
private void OnExternalServiceDataReceived(object sender, DataEventArgs e)
{
    // Event may fire on background thread
    _dispatcherQueue.TryEnqueue(() =>
    {
        DataList.ItemsSource = e.Data;
    });
}
```

**Common Mistakes**:
```csharp
// WRONG: CoreDispatcher is UWP - not available in WinUI 3 desktop
await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => { });

// WRONG: Direct UI access from background thread
await Task.Run(() =>
{
    ResultText.Text = "Done"; // Throws exception!
});
```

**Reference**: https://learn.microsoft.com/en-us/windows/windows-app-sdk/api/winrt/microsoft.ui.dispatching.dispatcherqueue

---

## ASYNC-005: Implement Incremental Loading for Large Collections

**Rule**: For collections that could exceed 50 items, implement `ISupportIncrementalLoading` to load data as the user scrolls.

**Why**: Loading thousands of items upfront causes multi-second delays and high memory usage. Incremental loading shows content immediately and loads more on demand.

**Example (Correct)**:
```csharp
public class IncrementalItemsSource<T> : ObservableCollection<T>, ISupportIncrementalLoading
{
    private readonly Func<int, int, Task<IEnumerable<T>>> _loadMoreFunc;
    private int _currentPage = 0;
    private const int PageSize = 50;

    public bool HasMoreItems { get; private set; } = true;

    public IncrementalItemsSource(Func<int, int, Task<IEnumerable<T>>> loadMoreFunc)
    {
        _loadMoreFunc = loadMoreFunc;
    }

    public IAsyncOperation<LoadMoreItemsResult> LoadMoreItemsAsync(uint count)
    {
        return LoadMoreAsync().AsAsyncOperation();
    }

    private async Task<LoadMoreItemsResult> LoadMoreAsync()
    {
        var items = await _loadMoreFunc(_currentPage, PageSize);
        var itemList = items.ToList();

        foreach (var item in itemList)
        {
            Add(item);
        }

        _currentPage++;
        HasMoreItems = itemList.Count == PageSize;

        return new LoadMoreItemsResult { Count = (uint)itemList.Count };
    }
}
```

```csharp
// Usage in ViewModel
public IncrementalItemsSource<Product> Products { get; }

public MainViewModel(IProductService productService)
{
    Products = new IncrementalItemsSource<Product>(
        (page, size) => productService.GetProductsAsync(page, size)
    );
}
```

**Common Mistakes**:
```csharp
// WRONG: Loads all items at once
public async Task LoadAllProductsAsync()
{
    var allProducts = await _service.GetAllProductsAsync(); // 10,000 items!
    Products = new ObservableCollection<Product>(allProducts);
}
```

**Uno Platform Notes**: `ISupportIncrementalLoading` is supported. For `ItemsRepeater`, use the Uno Toolkit `ItemsRepeaterExtensions.SupportsIncrementalLoading` attached property.

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.data.isupportincrementalloading

---

## ASYNC-006: Use ConfigureAwait(false) in Library Code

**Rule**: In non-UI library code, use `ConfigureAwait(false)` to avoid capturing the synchronization context.

**Why**: Capturing context has overhead and can cause deadlocks in certain scenarios. Library code shouldn't assume it needs UI thread access.

**Example (Correct)**:
```csharp
// In a service/library class (not UI code)
public class DataService
{
    public async Task<Data> GetDataAsync()
    {
        var json = await _httpClient.GetStringAsync(url).ConfigureAwait(false);
        var parsed = await ParseAsync(json).ConfigureAwait(false);
        return parsed;
    }
}
```

**When NOT to use ConfigureAwait(false)**:
```csharp
// In Page/UserControl code-behind - need UI context
public async Task LoadAsync()
{
    var data = await _service.GetDataAsync(); // Don't use ConfigureAwait(false)
    DataList.ItemsSource = data; // Need to be on UI thread
}
```

**Common Mistakes**:
```csharp
// WRONG: ConfigureAwait(false) in UI code causes exception
public async Task LoadAsync()
{
    var data = await _service.GetDataAsync().ConfigureAwait(false);
    DataList.ItemsSource = data; // Exception! Not on UI thread
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007

---

## ASYNC-007: Handle Async Void Properly

**Rule**: Use `async void` only for event handlers. All other async methods should return `Task` or `Task<T>`.

**Why**: `async void` methods can't be awaited, exceptions can't be caught by callers, and they can cause unobserved exceptions that crash the app.

**Example (Correct)**:
```csharp
// Event handler - async void is acceptable
private async void SaveButton_Click(object sender, RoutedEventArgs e)
{
    try
    {
        SaveButton.IsEnabled = false;
        await SaveDataAsync();
    }
    catch (Exception ex)
    {
        await ShowErrorDialogAsync(ex.Message);
    }
    finally
    {
        SaveButton.IsEnabled = true;
    }
}

// Regular method - return Task
public async Task SaveDataAsync()
{
    await _repository.SaveAsync(_data);
}
```

**Common Mistakes**:
```csharp
// WRONG: async void for non-event-handler
public async void LoadData() // Should return Task
{
    var data = await _service.GetDataAsync();
    ProcessData(data);
}

// WRONG: No exception handling in async void
private async void SaveButton_Click(object sender, RoutedEventArgs e)
{
    await SaveDataAsync(); // Exception crashes app!
}
```

**Reference**: https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
