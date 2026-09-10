# Data & HTTP Management (Medium Priority)

## HTTP Client Configuration with Uno.Extensions

**When to apply:** API integration

### Setup in App.xaml.cs
```csharp
protected override async void OnLaunched(LaunchActivatedEventArgs args)
{
    var builder = this.CreateBuilder(args)
        .Configure(host => host
            .UseConfiguration(configure: configBuilder =>
                configBuilder
                    .EmbeddedSource<App>()
                    .Section<AppConfig>()
            )
            .UseHttp((context, services) => {
#if DEBUG
                services.AddTransient<DelegatingHandler, DebugHttpHandler>();
#endif
                services.AddRefitClient<IWeatherApi>(context);
            })
        );

    MainWindow = builder.Window;
    Host = await builder.NavigateAsync<Shell>();
}
```

### Configuration File (appsettings.json)
```json
{
  "WeatherApi": {
    "Url": "https://api.openweathermap.org",
    "UseNativeHandler": true
  }
}
```

### Refit Interface
```csharp
public interface IWeatherApi
{
    [Get("/data/2.5/weather")]
    Task<WeatherResponse> GetWeatherAsync(
        [Query] string q,
        [Query] string appid,
        CancellationToken cancellationToken = default);
}
```

### Service Registration Extension
```csharp
public static IServiceCollection AddRefitClient<TInterface>(
    this IServiceCollection services,
    HostBuilderContext context)
    where TInterface : class
{
    var configSection = typeof(TInterface).Name.TrimStart('I');
    var config = context.Configuration.GetSection(configSection).Get<ApiConfig>();

    services.AddRefitClient<TInterface>()
        .ConfigureHttpClient(client => {
            client.BaseAddress = new Uri(config.Url);
            client.Timeout = TimeSpan.FromSeconds(30);
        })
        .ConfigurePrimaryHttpMessageHandler(() => new HttpClientHandler())
        .AddHttpMessageHandler<DebugHttpHandler>();

    return services;
}
```

### References
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Walkthrough/Refit.howto.html
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\App.xaml.cs

## Debug HTTP Handler

**When to apply:** Development/debugging

### Implementation
```csharp
internal class DebugHttpHandler : DelegatingHandler
{
    public DebugHttpHandler(HttpMessageHandler? innerHandler = null)
        : base(innerHandler ?? new HttpClientHandler())
    {
    }

    protected async override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        var response = await base.SendAsync(request, cancellationToken);

#if DEBUG
        if (!response.IsSuccessStatusCode)
        {
            Console.Error.WriteLine("Unsuccessful API Call");
            Console.Error.WriteLine($"{request.RequestUri} ({request.Method})");

            foreach ((var key, var values) in request.Headers.ToDictionary())
            {
                Console.Error.WriteLine($"  {key}: {string.Join(", ", values)}");
            }

            var content = request.Content is not null
                ? await request.Content.ReadAsStringAsync()
                : null;

            if (!string.IsNullOrEmpty(content))
            {
                Console.Error.WriteLine(content);
            }
        }
#endif

        return response;
    }
}
```

### Registration
```csharp
.UseHttp((context, services) => {
#if DEBUG
    services.AddTransient<DelegatingHandler, DebugHttpHandler>();
#endif
})
```

### References
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\Services\Endpoints\DebugHandler.cs

## LiteDB for Local Storage

**When to apply:** Local database needs

### Setup
```xml
<ItemGroup>
  <PackageReference Include="LiteDB" Version="5.0.17" />
</ItemGroup>
```

### Model
```csharp
using LiteDB;

public class Note
{
    [BsonId]
    public int Id { get; set; }

    public string Title { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
    public float[]? Embedding { get; set; }  // For vector search
}
```

### Database Service
```csharp
public class DatabaseService : IDisposable
{
    private readonly LiteDatabase _database;

    public DatabaseService(string databasePath)
    {
        _database = new LiteDatabase(databasePath);
    }

    public int InsertNote(Note note)
    {
        var collection = _database.GetCollection<Note>("notes");
        return collection.Insert(note);
    }

    public List<Note> GetAllNotes()
    {
        var collection = _database.GetCollection<Note>("notes");
        return collection.Query()
            .OrderByDescending(x => x.CreatedAt)
            .ToList();
    }

    public Note? GetNoteById(int id)
    {
        var collection = _database.GetCollection<Note>("notes");
        return collection.FindById(id);
    }

    public bool UpdateNote(Note note)
    {
        var collection = _database.GetCollection<Note>("notes");
        return collection.Update(note);
    }

    public bool DeleteNote(int id)
    {
        var collection = _database.GetCollection<Note>("notes");
        return collection.Delete(id);
    }

    public List<Note> SearchSimilarNotes(float[] queryEmbedding, int limit = 10)
    {
        var collection = _database.GetCollection<Note>("notes");
        return collection.Query()
            .OrderBySimilarity(x => x.Embedding, queryEmbedding)
            .Limit(limit)
            .ToList();
    }

    public void EnsureVectorIndex(int dimensions)
    {
        var collection = _database.GetCollection<Note>("notes");
        collection.EnsureIndex(x => x.Embedding, new VectorIndexOptions(dimensions));
    }

    public void Dispose()
    {
        _database?.Dispose();
    }
}
```

### DI Registration
```csharp
var dbPath = Path.Combine(
    Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
    "app.db");

services.AddSingleton(new DatabaseService(dbPath));
```

### References
- Example: C:\Users\Platform006\source\repos\SmartNotes\SmartNotes\Services\DatabaseService.cs

## Serialization Configuration

**When to apply:** JSON serialization

### Setup in App.xaml.cs
```csharp
.UseConfiguration(configure: configBuilder =>
    configBuilder
        .EmbeddedSource<App>()
        .Section<AppConfig>()
)
```

### Configuration Model
```csharp
public record AppConfig
{
    public string? Environment { get; init; }
    public string? ApiKey { get; init; }
    public ApiEndpoints? Endpoints { get; init; }
}

public record ApiEndpoints
{
    public string? Weather { get; init; }
    public string? Maps { get; init; }
}
```

### appsettings.json as Embedded Resource
```xml
<ItemGroup>
  <EmbeddedResource Include="appsettings.json" />
  <EmbeddedResource Include="appsettings.Development.json" />
</ItemGroup>
```

### Accessing Configuration
```csharp
public class MyService
{
    private readonly AppConfig _config;

    public MyService(IOptions<AppConfig> config)
    {
        _config = config.Value;
    }

    public async Task DoWork()
    {
        var apiUrl = _config.Endpoints?.Weather;
        // Use apiUrl
    }
}
```

### References
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Configuration/ConfigurationOverview.html
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\App.xaml.cs

## Error Handling Pattern

**When to apply:** All async operations

### ViewModel Pattern
```csharp
public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private bool _isLoading;

    [ObservableProperty]
    private string? _errorMessage;

    [RelayCommand]
    private async Task LoadDataAsync()
    {
        IsLoading = true;
        ErrorMessage = null;

        try
        {
            var data = await _apiService.GetDataAsync();
            // Process data
        }
        catch (HttpRequestException ex)
        {
            ErrorMessage = $"Network error: {ex.Message}";
        }
        catch (Exception ex)
        {
            ErrorMessage = $"Error: {ex.Message}";
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

### XAML Display
```xml
<StackPanel>
    <ProgressRing IsActive="{x:Bind ViewModel.IsLoading, Mode=OneWay}" />

    <TextBlock Text="{x:Bind ViewModel.ErrorMessage, Mode=OneWay}"
               Foreground="{ThemeResource ErrorColor}"
               Visibility="{x:Bind ViewModel.ErrorMessage, Converter={StaticResource NullToCollapsedConverter}, Mode=OneWay}" />
</StackPanel>
```

### MVUX Pattern (Automatic)
```csharp
public partial record DataModel(IDataService DataService)
{
    public IListFeed<Item> Items => ListFeed.Async(DataService.GetItems);
}
```

MVUX automatically provides:
- `Items.IsExecuting` (loading state)
- `Items.Error` (exception if any)
- Automatic retry via refresh

### References
- Pattern from multiple examples across repositories
