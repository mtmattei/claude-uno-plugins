---
title: ISupportIncrementalLoading for Anticipatory Paging
impact: MEDIUM
tags: prefetch, incremental, paging, listview
---

## ISupportIncrementalLoading for Anticipatory Paging

Implement ISupportIncrementalLoading on your collection so ListView automatically loads the next page before the user scrolls to the end, rather than showing a "load more" button.

**Incorrect (manual load-more button — interrupts scroll flow):**

```xml
<StackPanel>
    <ListView ItemsSource="{x:Bind Items}" />
    <Button Content="Load More" Click="OnLoadMore" />
</StackPanel>
```

**Correct (incremental loading — seamless infinite scroll):**

```csharp
public class IncrementalItemSource : ObservableCollection<Item>,
    ISupportIncrementalLoading
{
    public bool HasMoreItems => _hasMore;

    public IAsyncOperation<LoadMoreItemsResult> LoadMoreItemsAsync(uint count)
    {
        return AsyncInfo.Run(async ct =>
        {
            var items = await _service.GetPageAsync(_page++, (int)count, ct);
            foreach (var item in items) Add(item);
            _hasMore = items.Count == count;
            return new LoadMoreItemsResult { Count = (uint)items.Count };
        });
    }
}
```

```xml
<ListView ItemsSource="{x:Bind IncrementalItems}"
          IncrementalLoadingTrigger="Edge"
          DataFetchSize="2" />
```
