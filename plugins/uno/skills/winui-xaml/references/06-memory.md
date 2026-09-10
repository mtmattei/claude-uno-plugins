# Memory Management Skills - WinUI 3 XAML

## MEMORY-001: Unsubscribe Event Handlers in Unloaded

**Rule**: Always unsubscribe from events in the `Unloaded` event handler. This includes service events, application events, and timer events.

**Why**: Event subscriptions create strong references that prevent garbage collection. Pages/controls that subscribe without unsubscribing cause memory leaks - the objects are never collected even after navigation.

**Example (Correct)**:
```csharp
public sealed partial class MyPage : Page
{
    private readonly IDataService _dataService;
    private readonly DispatcherQueueTimer _timer;

    public MyPage()
    {
        InitializeComponent();

        _dataService = App.Services.GetService<IDataService>();
        _dataService.DataChanged += OnDataChanged;

        _timer = DispatcherQueue.GetForCurrentThread().CreateTimer();
        _timer.Tick += OnTimerTick;

        // Subscribe to Unloaded for cleanup
        Unloaded += OnUnloaded;
    }

    private void OnUnloaded(object sender, RoutedEventArgs e)
    {
        // Unsubscribe from all events
        _dataService.DataChanged -= OnDataChanged;
        _timer.Tick -= OnTimerTick;
        _timer.Stop();

        Unloaded -= OnUnloaded;
    }

    private void OnDataChanged(object sender, EventArgs e) { /* ... */ }
    private void OnTimerTick(DispatcherQueueTimer sender, object e) { /* ... */ }
}
```

**Common Mistakes**:
```csharp
// WRONG: No cleanup - memory leak!
public sealed partial class MyPage : Page
{
    public MyPage()
    {
        InitializeComponent();

        var service = App.Services.GetService<IDataService>();
        service.DataChanged += OnDataChanged; // Never unsubscribed

        Application.Current.Suspending += OnSuspending; // Never unsubscribed
    }
}
```

**Common Leak Sources**:
- Static event handlers
- Application.Current events (Suspending, Resuming)
- Service/repository events
- Timer events
- Observable collection events
- Messaging/event aggregator subscriptions

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/memory-leaks

---

## MEMORY-002: Use WeakReference for Long-lived Subscriptions

**Rule**: For long-lived objects that subscribe to events from shorter-lived objects, use `WeakEventListener` or weak references.

**Why**: Strong references from long-lived services to UI elements prevent UI garbage collection. Weak references allow collection while the subscription is active.

**Example (Correct with CommunityToolkit)**:
```csharp
using CommunityToolkit.WinUI;

public sealed partial class MyPage : Page
{
    private WeakEventListener<MyPage, IDataService, EventArgs> _weakListener;

    public MyPage()
    {
        InitializeComponent();

        var service = App.Services.GetService<IDataService>();

        _weakListener = new WeakEventListener<MyPage, IDataService, EventArgs>(this)
        {
            OnEventAction = static (instance, source, args) => instance.OnDataChanged(source, args),
            OnDetachAction = static (listener) =>
            {
                if (listener.Source is IDataService svc)
                    svc.DataChanged -= listener.OnEvent;
            }
        };

        service.DataChanged += _weakListener.OnEvent;
    }

    private void OnDataChanged(object sender, EventArgs e)
    {
        // Handle event
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/api/system.weakreference

---

## MEMORY-003: Dispose IDisposable Resources

**Rule**: Dispose of resources that implement `IDisposable`: streams, HTTP clients, database connections, timers.

**Why**: Unmanaged resources are not garbage collected. Failure to dispose causes resource exhaustion - file handles, network connections, memory.

**Example (Correct)**:
```csharp
public sealed partial class ImageViewerPage : Page, IDisposable
{
    private MemoryStream _imageStream;
    private HttpClient _httpClient;
    private bool _disposed;

    public ImageViewerPage()
    {
        InitializeComponent();
        _httpClient = new HttpClient();
        Unloaded += (s, e) => Dispose();
    }

    public async Task LoadImageAsync(string url)
    {
        var bytes = await _httpClient.GetByteArrayAsync(url);
        _imageStream?.Dispose(); // Dispose previous
        _imageStream = new MemoryStream(bytes);
        // Use stream...
    }

    public void Dispose()
    {
        if (_disposed) return;
        _disposed = true;

        _imageStream?.Dispose();
        _httpClient?.Dispose();
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Streams not disposed
public async Task ProcessFilesAsync()
{
    foreach (var file in files)
    {
        var stream = await file.OpenStreamForReadAsync();
        ProcessStream(stream);
        // stream never disposed!
    }
}
```

**Use `using` for automatic disposal**:
```csharp
public async Task ProcessFilesAsync()
{
    foreach (var file in files)
    {
        await using var stream = await file.OpenStreamForReadAsync();
        await ProcessStreamAsync(stream);
    } // Disposed automatically
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose

---

## MEMORY-004: Clear References to Large Objects

**Rule**: Set references to large objects (images, collections, data) to null when no longer needed, especially before navigation.

**Why**: Helps garbage collector identify objects for collection sooner. Critical for memory-intensive objects like bitmaps.

**Example (Correct)**:
```csharp
public sealed partial class GalleryPage : Page
{
    private List<BitmapImage> _images;

    protected override void OnNavigatedFrom(NavigationEventArgs e)
    {
        base.OnNavigatedFrom(e);

        // Clear image references to allow GC
        foreach (var image in _images ?? Enumerable.Empty<BitmapImage>())
        {
            image.UriSource = null;
        }
        _images?.Clear();
        _images = null;

        // Clear ItemsSource
        ImageGrid.ItemsSource = null;

        // Force cleanup if needed (use sparingly)
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/reduce-memory-usage

---

## MEMORY-005: Avoid Closures Capturing `this`

**Rule**: Be cautious with lambdas and closures that capture `this` in event handlers or async callbacks. They create implicit references.

**Why**: Closures that capture `this` create strong references that can prevent garbage collection of the containing object.

**Example (Correct)**:
```csharp
public sealed partial class MyPage : Page
{
    public MyPage()
    {
        InitializeComponent();

        // Capture only what's needed, not 'this'
        var weakThis = new WeakReference<MyPage>(this);

        SomeService.OnComplete += (s, e) =>
        {
            if (weakThis.TryGetTarget(out var page))
            {
                page.UpdateUI(e.Result);
            }
        };
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Lambda captures 'this', preventing GC
public MyPage()
{
    LongLivedService.OnComplete += (s, e) =>
    {
        this.UpdateUI(e.Result); // 'this' captured - page can't be GC'd
    };
}
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions

---

## MEMORY-006: Use Object Pooling for Frequent Allocations

**Rule**: For objects created frequently (per-frame, per-scroll), use object pooling to reduce allocation pressure.

**Why**: Frequent allocations cause GC pressure, leading to GC pauses that cause UI jank. Pooling reuses objects instead of allocating new ones.

**Example (Correct)**:
```csharp
using Microsoft.Extensions.ObjectPool;

public class ItemRendererPool
{
    private readonly ObjectPool<RenderItem> _pool;

    public ItemRendererPool()
    {
        var policy = new DefaultPooledObjectPolicy<RenderItem>();
        _pool = new DefaultObjectPool<RenderItem>(policy, 100);
    }

    public RenderItem Rent()
    {
        return _pool.Get();
    }

    public void Return(RenderItem item)
    {
        item.Reset(); // Clear state before returning
        _pool.Return(item);
    }
}

public class RenderItem
{
    public string Text { get; set; }
    public Rect Bounds { get; set; }

    public void Reset()
    {
        Text = null;
        Bounds = default;
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Allocates new object every frame
private void OnRender()
{
    foreach (var data in _dataItems)
    {
        var renderItem = new RenderItem // Allocation per item per frame!
        {
            Text = data.Text,
            Bounds = CalculateBounds(data)
        };
        Render(renderItem);
    }
}
```

**Reference**: https://learn.microsoft.com/en-us/aspnet/core/performance/objectpool

---

## MEMORY-007: Monitor Memory with Diagnostics

**Rule**: Use memory profiling tools to identify leaks during development. Monitor `AppMemoryUsage` in release builds.

**Why**: Memory issues are often invisible until they cause crashes. Proactive monitoring catches leaks early.

**Example (Correct)**:
```csharp
#if DEBUG
public static class MemoryDiagnostics
{
    public static void LogMemoryUsage(string context)
    {
        var memoryUsage = MemoryManager.AppMemoryUsage;
        var memoryLimit = MemoryManager.AppMemoryUsageLimit;
        var percentage = (double)memoryUsage / memoryLimit * 100;

        Debug.WriteLine($"[Memory] {context}: {memoryUsage / 1024 / 1024:F1} MB " +
                        $"({percentage:F1}% of {memoryLimit / 1024 / 1024:F0} MB limit)");
    }
}
#endif

// Usage
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    #if DEBUG
    MemoryDiagnostics.LogMemoryUsage($"Navigated to {GetType().Name}");
    #endif
}
```

**Reference**: https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/analyze-memory-usage
