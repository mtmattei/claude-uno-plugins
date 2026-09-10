# Memory and Resource Management

## MEMORY-001: Dispose Resources Properly with Using Statements

**Impact**: HIGH (Prevents memory leaks and resource exhaustion)

Not disposing `IDisposable` resources leads to memory leaks, connection pool exhaustion, and file handle leaks. These issues compound over time and cause production outages.

### Incorrect

```csharp
// ❌ BAD: Not disposing HttpClient (creates new sockets each time)
public class ApiClient
{
    public async Task<string> GetData()
    {
        var client = new HttpClient(); // Socket exhaustion!
        return await client.GetStringAsync("https://api.example.com/data");
    }
}

// ❌ BAD: Not disposing streams
public async Task ProcessFile(string path)
{
    var stream = File.OpenRead(path);
    await ProcessStream(stream);
    // Stream handle never released!
}

// ❌ BAD: Not disposing database connections
public async Task<User> GetUser(int id)
{
    var connection = new SqlConnection(connectionString);
    await connection.OpenAsync();
    // ... query logic
    // Connection never returned to pool!
}
```

### Correct

```csharp
// ✅ GOOD: Use IHttpClientFactory for HttpClient
builder.Services.AddHttpClient<IApiClient, ApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.Timeout = TimeSpan.FromSeconds(30);
});

public class ApiClient : IApiClient
{
    private readonly HttpClient _httpClient;

    public ApiClient(HttpClient httpClient)
    {
        _httpClient = httpClient; // Managed by factory
    }

    public async Task<string> GetData()
    {
        return await _httpClient.GetStringAsync("/data");
    }
}

// ✅ GOOD: Use using statement for streams
public async Task ProcessFile(string path)
{
    await using var stream = File.OpenRead(path);
    await ProcessStream(stream);
    // Stream automatically disposed
}

// ✅ GOOD: DbContext manages connections automatically
public class UserService
{
    private readonly AppDbContext _context;

    public UserService(AppDbContext context)
    {
        _context = context; // Scoped lifetime handles disposal
    }

    public async Task<User> GetUser(int id)
    {
        return await _context.Users.FindAsync(id);
        // Connection returned to pool automatically
    }
}

// ✅ GOOD: Implement IDisposable correctly
public class ResourceManager : IDisposable
{
    private readonly DbConnection _connection;
    private bool _disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;

        if (disposing)
        {
            _connection?.Dispose();
        }

        _disposed = true;
    }
}
```

### Additional Context

Use `using` declarations (C# 8+) for cleaner code when scope is the entire method:
```csharp
public async Task ProcessFile(string path)
{
    using var stream = File.OpenRead(path); // Disposed at method end
    await ProcessStream(stream);
}
```

For async disposal, implement `IAsyncDisposable` and use `await using`.

### Reference

- [IDisposable Pattern](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose)
- [Using Statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)
- [IHttpClientFactory Best Practices](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)
