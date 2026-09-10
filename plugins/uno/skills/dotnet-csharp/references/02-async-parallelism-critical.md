# Async and Parallelism Patterns

## ASYNC-001: Eliminate Async Waterfalls with Parallel Execution

**Impact**: CRITICAL (Can cut response times by 50% or more)

Async waterfalls occur when independent async operations are awaited sequentially, forcing each to complete before the next starts. This multiplies latency instead of overlapping work.

### Incorrect

```csharp
// ❌ BAD: Sequential execution when operations are independent
public async Task<DashboardData> GetDashboard(int userId)
{
    var user = await GetUser(userId);          // Wait 50ms
    var orders = await GetOrders(userId);      // Wait 100ms
    var recommendations = await GetRecs(userId); // Wait 150ms

    return new DashboardData(user, orders, recommendations);
    // Total time: 50 + 100 + 150 = 300ms
}
```

### Correct

```csharp
// ✅ GOOD: Parallel execution for independent operations
public async Task<DashboardData> GetDashboard(int userId)
{
    var userTask = GetUser(userId);
    var ordersTask = GetOrders(userId);
    var recsTask = GetRecs(userId);

    await Task.WhenAll(userTask, ordersTask, recsTask);

    return new DashboardData(
        await userTask,
        await ordersTask,
        await recsTask
    );
    // Total time: max(50, 100, 150) = 150ms (50% reduction)
}
```

### Additional Context

Only parallelize truly independent operations. If one operation depends on the result of another, keep them sequential. Use `Task.WhenAny` when you need the first completed result, or `Parallel.ForEachAsync` for processing collections concurrently.

### Reference

- [Async/Await Best Practices](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- [Task.WhenAll Documentation](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall)
