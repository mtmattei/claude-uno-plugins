# Database Access Patterns

## DB-001: Prevent N+1 Queries with Eager Loading

**Impact**: CRITICAL (Can reduce database round trips from thousands to single digits)

The N+1 query problem occurs when lazy loading triggers separate database queries for each related entity. This can turn a single query into hundreds or thousands of queries, devastating performance.

### Incorrect

```csharp
// ❌ BAD: Triggers separate query for each order's items
var orders = await context.Orders.ToListAsync();
foreach (var order in orders)
{
    // Each access to Items triggers a new query
    var itemCount = order.Items.Count();
    var total = order.Items.Sum(i => i.Price);
}
// Result: 1 query for orders + N queries for items = N+1 queries
```

### Correct

```csharp
// ✅ GOOD: Single query with JOIN
var orders = await context.Orders
    .Include(o => o.Items)
    .ToListAsync();

foreach (var order in orders)
{
    var itemCount = order.Items.Count();
    var total = order.Items.Sum(i => i.Price);
}
// Result: 1 query with JOIN

// ✅ BETTER: Include multiple levels
var orders = await context.Orders
    .Include(o => o.Items)
        .ThenInclude(i => i.Product)
    .Include(o => o.Customer)
    .ToListAsync();
```

### Additional Context

For complex scenarios with multiple includes, consider using:
- `AsSplitQuery()` if loading many collections to avoid cartesian explosion
- Projections with `Select()` if you only need specific fields
- `AsNoTracking()` for read-only queries to further improve performance

### Reference

- [Entity Framework Core - Loading Related Data](https://learn.microsoft.com/en-us/ef/core/querying/related-data/)
- [Performance - N+1 Queries](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying#beware-of-lazy-loading)
