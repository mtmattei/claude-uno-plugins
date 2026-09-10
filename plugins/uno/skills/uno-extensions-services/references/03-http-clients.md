# HTTP Clients

## HTTP-001: Generate Kiota Clients from OpenAPI Specifications

**Rule**: Use the Kiota CLI to generate strongly-typed C# clients from OpenAPI/Swagger specs, then register them with `AddKiotaClient<T>()` via `UseHttp()`.

**Why**: Kiota generates compile-time-safe API clients with full request/response models, eliminating hand-written DTOs and HTTP plumbing. The generated client integrates with `HttpClientFactory` lifecycle management, and when combined with `Uno.Extensions.Authentication`, the `Authorization` header is automatically injected. Manual `HttpClient` construction misses retry policies, native handlers, and token injection.

**Example (CLI)**:
```bash
# Install Kiota CLI
dotnet tool install --global Microsoft.OpenApi.Kiota

# Generate client from a local spec file
kiota generate \
    --openapi ./specs/petstore.json \
    --language CSharp \
    --class-name PetStoreClient \
    --namespace-name MyApp.Client.PetStore \
    --output ./MyApp/Client/PetStore

# Or generate from a running server's Swagger endpoint
kiota generate \
    --openapi http://localhost:5002/swagger/v1/swagger.json \
    --language CSharp \
    --class-name MyApiClient \
    --namespace-name MyApp.Client.MyApi \
    --output ./MyApp/Client/MyApi
```

**Example (C# - Registration)**:
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args)
        .Configure(host =>
        {
            host.UseHttp((context, services) =>
                services.AddKiotaClient<PetStoreClient>(
                    context,
                    options: new EndpointOptions { Url = "https://petstore.example.com" }
                )
            );
        });
}
```

**Example (C# - Usage in ViewModel)**:
```csharp
public class PetsViewModel
{
    private readonly PetStoreClient _client;

    public PetsViewModel(PetStoreClient client)
    {
        _client = client;
    }

    public async Task<List<Pet>?> GetPetsAsync()
    {
        return await _client.Pets.GetAsync();
    }
}
```

**Common Mistakes**:
```bash
# WRONG: Forgetting to add HttpKiota to UnoFeatures
<UnoFeatures>
    Extensions;
    Http;           <!-- Http alone does not include Kiota support -->
</UnoFeatures>
# Correct: Add HttpKiota
<UnoFeatures>
    Extensions;
    HttpKiota;
</UnoFeatures>
```

```csharp
// WRONG: Manually creating HttpClient for a Kiota client - loses DI lifecycle
var httpClient = new HttpClient { BaseAddress = new Uri("https://petstore.example.com") };
var client = new PetStoreClient(new HttpClientRequestAdapter(
    new AnonymousAuthenticationProvider(), httpClient: httpClient));
// Misses native handler, auth token injection, and HttpClientFactory pooling
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Http/HowTo-Kiota.html

---

## HTTP-002: Define Refit Interfaces for Typed REST Clients

**Rule**: Define a C# interface with Refit attributes (`[Get]`, `[Post]`, etc.) and register it with `AddRefitClient<TInterface>(context)` inside `UseHttp()`.

**Why**: Refit generates the HTTP implementation at compile time from your interface, reducing boilerplate while maintaining type safety. The `AddRefitClient<T>` extension auto-discovers endpoint configuration from `appsettings.json` using the interface name (minus the leading `I`) as the section key. This keeps URL and handler configuration declarative and environment-specific.

**Example (C# - Interface)**:
```csharp
public interface IChuckNorrisEndpoint
{
    [Get("/jokes/random")]
    Task<JokeResponse> GetRandomJokeAsync(CancellationToken ct = default);

    [Get("/jokes/search")]
    Task<SearchResult> SearchJokesAsync(
        [Query] string query,
        CancellationToken ct = default);
}
```

**Example (JSON - appsettings.json)**:
```json
{
  "ChuckNorrisEndpoint": {
    "Url": "https://api.chucknorris.io/",
    "UseNativeHandler": true
  }
}
```

**Example (C# - Registration)**:
```csharp
host.UseHttp((context, services) =>
{
    services.AddRefitClient<IChuckNorrisEndpoint>(context);
});
```

**Example (C# - Usage)**:
```csharp
public class JokesViewModel
{
    private readonly IChuckNorrisEndpoint _api;

    public JokesViewModel(IChuckNorrisEndpoint api)
    {
        _api = api;
    }

    public async Task<string> GetJokeAsync()
    {
        var response = await _api.GetRandomJokeAsync();
        return response.Value;
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Configuration section name does not match interface name
// Interface: IWeatherApi -> expected section: "WeatherApi"
{
  "Weather": {           // Should be "WeatherApi" (interface name minus "I")
    "Url": "https://..."
  }
}
// Result: Url is null, HttpClient has no BaseAddress, requests fail

// WRONG: Adding HttpRefit instead of the correct UnoFeature
<UnoFeatures>
    Http;               <!-- Base HTTP only - no Refit support -->
</UnoFeatures>
// Correct:
<UnoFeatures>
    HttpRefit;
</UnoFeatures>
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Http/HttpOverview.html#refit

---

## HTTP-003: Configure Endpoint Options in appsettings.json

**Rule**: Define HTTP endpoint URLs and handler settings in `appsettings.json` sections that match the service or client name. Enable `UseNativeHandler` for production performance.

**Why**: Centralizing endpoint configuration in `appsettings.json` enables environment-specific overrides (dev vs staging vs production) without code changes. The `UseNativeHandler` flag tells the HTTP layer to use the platform's native HTTP stack (NSUrlSession on iOS, AndroidClientHandler on Android), which provides better performance, TLS support, and proxy handling than the managed .NET handler.

**Example (JSON - appsettings.json)**:
```json
{
  "WeatherApi": {
    "Url": "https://api.openweathermap.org",
    "UseNativeHandler": true
  },
  "PaymentGateway": {
    "Url": "https://payments.example.com/v2",
    "UseNativeHandler": true
  }
}
```

**Example (JSON - appsettings.development.json)**:
```json
{
  "WeatherApi": {
    "Url": "https://localhost:5001",
    "UseNativeHandler": false
  },
  "PaymentGateway": {
    "Url": "https://sandbox.payments.example.com/v2",
    "UseNativeHandler": false
  }
}
```

**Example (C# - Inline EndpointOptions when not using config)**:
```csharp
host.UseHttp((context, services) =>
{
    services.AddClient<IShowService, ShowService>(context,
        new EndpointOptions
        {
            Url = "https://ch9-app.azurewebsites.net/"
        }
        .Enable(nameof(EndpointOptions.UseNativeHandler)));
});
```

**Common Mistakes**:
```json
// WRONG: appsettings.json not marked as EmbeddedResource - file not found at runtime
// Ensure the .csproj contains:
<EmbeddedResource Include="appsettings.json" />
<EmbeddedResource Include="appsettings.development.json" />

// WRONG: Using UseNativeHandler: true during local development with self-signed certs
// Native handlers enforce strict TLS validation - self-signed certs will fail
// Use UseNativeHandler: false in appsettings.development.json
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Http/HttpOverview.html#register-endpoints

---

## HTTP-004: Implement a DebugHttpHandler for Development Diagnostics

**Rule**: Create a `DelegatingHandler` that logs request/response details and register it conditionally under `#if DEBUG`.

**Why**: HTTP debugging on mobile and WebAssembly targets lacks browser DevTools or Fiddler. A `DelegatingHandler` intercepts every request in the pipeline, logging URLs, headers, status codes, and response bodies. Wrapping it in `#if DEBUG` ensures zero overhead in release builds. This handler slots into the `HttpClientFactory` pipeline alongside auth and retry handlers.

**Example (C#)**:
```csharp
internal class DebugHttpHandler : DelegatingHandler
{
    private readonly ILogger<DebugHttpHandler> _logger;

    public DebugHttpHandler(ILogger<DebugHttpHandler> logger)
    {
        _logger = logger;
    }

    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        _logger.LogDebug("HTTP {Method} {Uri}", request.Method, request.RequestUri);

        var response = await base.SendAsync(request, cancellationToken);

        if (!response.IsSuccessStatusCode)
        {
            var body = response.Content is not null
                ? await response.Content.ReadAsStringAsync(cancellationToken)
                : "(empty)";
            _logger.LogWarning(
                "HTTP {StatusCode} from {Uri}: {Body}",
                (int)response.StatusCode,
                request.RequestUri,
                body);
        }

        return response;
    }
}
```

**Example (C# - Registration)**:
```csharp
host.UseHttp((context, services) =>
{
#if DEBUG
    services.AddTransient<DelegatingHandler, DebugHttpHandler>();
#endif
    services.AddRefitClient<IWeatherApi>(context);
});
```

**Common Mistakes**:
```csharp
// WRONG: Registering debug handler without #if DEBUG - logs sensitive data in production
services.AddTransient<DelegatingHandler, DebugHttpHandler>();

// WRONG: Reading response body without buffering - stream consumed, downstream fails
protected override async Task<HttpResponseMessage> SendAsync(...)
{
    var response = await base.SendAsync(request, cancellationToken);
    var body = await response.Content.ReadAsStringAsync(); // Consumes stream
    Console.WriteLine(body);
    return response; // Content stream is now empty for the caller
}
// Fix: Use response.Content.LoadIntoBufferAsync() first, or read after copying
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Http/HttpOverview.html

---

## HTTP-005: Let HttpClientFactory Manage HttpClient Lifetimes

**Rule**: Never instantiate `new HttpClient()` directly. Always resolve HTTP clients through DI via `AddClient`, `AddRefitClient`, or `AddKiotaClient` which use `HttpClientFactory` internally.

**Why**: `HttpClientFactory` pools `HttpMessageHandler` instances to avoid socket exhaustion (`SocketException: Address already in use`) while periodically recycling them to pick up DNS changes. Manually creating `HttpClient` instances either leaks sockets (if you create many) or misses DNS updates (if you reuse a static instance). The factory also enables the handler pipeline (auth, retry, logging).

**Example (C#)**:
```csharp
// CORRECT: Let DI provide the client - HttpClientFactory manages the handler
public class WeatherService : IWeatherService
{
    private readonly HttpClient _httpClient;

    public WeatherService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<WeatherData?> GetForecastAsync(string city)
    {
        var response = await _httpClient.GetAsync($"/forecast?city={city}");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<WeatherData>();
    }
}

// Registration with typed client
host.UseHttp((context, services) =>
{
    services.AddClient<IWeatherService, WeatherService>(context, "WeatherApi");
});
```

**Common Mistakes**:
```csharp
// WRONG: new HttpClient() per request - socket exhaustion
public async Task<string> FetchDataAsync()
{
    using var client = new HttpClient(); // Creates new socket each time
    return await client.GetStringAsync("https://api.example.com/data");
}

// WRONG: Static HttpClient - never picks up DNS changes
private static readonly HttpClient _client = new HttpClient
{
    BaseAddress = new Uri("https://api.example.com")
};
// If the server IP changes, the static client keeps using the stale address
```

**Reference**: https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests

---

## HTTP-006: Use Custom EndpointOptions for Extended Configuration

**Rule**: Subclass `EndpointOptions` when you need additional per-endpoint settings (API keys, timeouts, custom headers) that should be configurable from `appsettings.json`.

**Why**: The base `EndpointOptions` only provides `Url` and `UseNativeHandler`. Real-world APIs often require API keys, custom headers, or per-client timeouts. Subclassing keeps all endpoint configuration in a single, typed section that is environment-overridable and injectable.

**Example (C#)**:
```csharp
public class WeatherEndpointOptions : EndpointOptions
{
    public string? ApiKey { get; set; }
    public int TimeoutSeconds { get; set; } = 30;
}
```

**Example (JSON - appsettings.json)**:
```json
{
  "WeatherApi": {
    "Url": "https://api.openweathermap.org",
    "UseNativeHandler": true,
    "ApiKey": "your-api-key-here",
    "TimeoutSeconds": 15
  }
}
```

**Example (C# - Registration)**:
```csharp
host.UseHttp((context, services) =>
{
    services.AddClientWithEndpoint<IWeatherService, WeatherService, WeatherEndpointOptions>();
});
```

**Common Mistakes**:
```csharp
// WRONG: Hardcoding API keys in service constructors
public class WeatherService : IWeatherService
{
    private const string ApiKey = "abc123"; // Compiled into binary, not overridable
}

// WRONG: Reading configuration manually instead of using typed options
var apiKey = configuration["WeatherApi:ApiKey"]; // Stringly-typed, no compile-time safety
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Http/HttpOverview.html#custom-endpoint-options

---

## HTTP-007: Handle HTTP Errors with Structured Patterns

**Rule**: Wrap API calls in try/catch blocks that distinguish network errors (`HttpRequestException`), API errors (non-success status codes), and deserialization failures. Surface user-friendly messages while logging full details.

**Why**: Mobile and cross-platform apps face intermittent connectivity, server errors, and unexpected response formats. Unhandled HTTP exceptions crash the app. Catching only the base `Exception` hides the root cause. Structured error handling lets you show "No internet connection" vs "Server error" vs "Unexpected response" and log the right details for debugging.

**Example (C#)**:
```csharp
public class WeatherViewModel : ObservableObject
{
    private readonly IWeatherApi _api;
    private readonly ILogger<WeatherViewModel> _logger;

    [ObservableProperty]
    private string? _errorMessage;

    [ObservableProperty]
    private bool _isLoading;

    [RelayCommand]
    private async Task LoadWeatherAsync()
    {
        IsLoading = true;
        ErrorMessage = null;

        try
        {
            var weather = await _api.GetCurrentAsync("Seattle");
            // Update UI with weather data
        }
        catch (HttpRequestException ex) when (ex.InnerException is SocketException)
        {
            ErrorMessage = "No internet connection. Please check your network.";
            _logger.LogWarning(ex, "Network connectivity error");
        }
        catch (HttpRequestException ex)
        {
            ErrorMessage = "Unable to reach the server. Please try again later.";
            _logger.LogError(ex, "HTTP request failed: {StatusCode}", ex.StatusCode);
        }
        catch (JsonException ex)
        {
            ErrorMessage = "Received an unexpected response from the server.";
            _logger.LogError(ex, "Failed to deserialize API response");
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Catching only Exception - no differentiation
catch (Exception ex)
{
    ErrorMessage = ex.Message; // Exposes internal details to users
}

// WRONG: No error handling at all - app crashes on network failure
var weather = await _api.GetCurrentAsync("Seattle"); // Unhandled exception
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Http/HttpOverview.html
