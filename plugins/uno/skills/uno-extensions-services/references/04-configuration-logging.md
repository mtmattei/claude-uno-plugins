# Configuration and Logging

## CONFIG-001: Use EmbeddedSource for appsettings.json on All Platforms

**Rule**: Register `appsettings.json` as an `EmbeddedResource` in the `.csproj` and load it with `.EmbeddedSource<App>()` in `UseConfiguration()`.

**Why**: Embedded resources are compiled into the assembly and available immediately at startup on every platform, including WebAssembly. The alternative `ContentSource<App>()` requires async file I/O which is unreliable on Wasm (the file may not be available until after the host builds). Using `EmbeddedSource` guarantees configuration is populated before any service that depends on `IConfiguration` or `IOptions<T>` is resolved.

**Example (XML - .csproj)**:
```xml
<ItemGroup>
    <EmbeddedResource Include="appsettings.json" />
    <EmbeddedResource Include="appsettings.development.json" />
    <EmbeddedResource Include="appsettings.production.json" />
</ItemGroup>
```

**Example (C#)**:
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args)
        .Configure(host =>
        {
            host.UseConfiguration(builder =>
                builder.EmbeddedSource<App>()
            );
        });

    Host = appBuilder.Build();
}
```

**Common Mistakes**:
```xml
<!-- WRONG: appsettings.json with default Content build action - not found on Wasm -->
<Content Include="appsettings.json" />
<!-- Fix: Change to EmbeddedResource -->

<!-- WRONG: File in wrong directory - EmbeddedSource looks in the assembly containing App -->
<!-- appsettings.json must be in the project root or a known embedded path -->
```

```csharp
// WRONG: Using ContentSource on WebAssembly
host.UseConfiguration(builder => builder.ContentSource<App>());
// On Wasm, the file may not be loaded before Host.Build() completes
// Configuration sections return null, services get default values
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Configuration/ConfigurationOverview.html#app-settings-file-sources

---

## CONFIG-002: Map Configuration Sections to Typed Classes with Section<T>

**Rule**: Use `.Section<T>()` to bind a JSON object in `appsettings.json` to a strongly-typed record or class, then inject it as `IOptions<T>`.

**Why**: Typed configuration sections provide compile-time safety, IntelliSense, and refactoring support. Raw `IConfiguration["key"]` access is stringly-typed, error-prone, and requires null checks at every call site. `Section<T>()` registers both `IOptions<T>` (read-only) and `IWritableOptions<T>` (writable) automatically.

**Example (JSON - appsettings.json)**:
```json
{
  "AppSettings": {
    "AppName": "My Weather App",
    "MaxRetries": 3,
    "CacheTimeoutMinutes": 15
  }
}
```

**Example (C# - Configuration class)**:
```csharp
public record AppSettings
{
    public string AppName { get; init; } = string.Empty;
    public int MaxRetries { get; init; } = 1;
    public int CacheTimeoutMinutes { get; init; } = 10;
}
```

**Example (C# - Registration)**:
```csharp
host.UseConfiguration(builder =>
    builder
        .EmbeddedSource<App>()
        .Section<AppSettings>()
);
```

**Example (C# - Injection)**:
```csharp
public class WeatherService : IWeatherService
{
    private readonly AppSettings _settings;

    public WeatherService(IOptions<AppSettings> settings)
    {
        _settings = settings.Value;
    }

    public TimeSpan CacheTimeout =>
        TimeSpan.FromMinutes(_settings.CacheTimeoutMinutes);
}
```

**Common Mistakes**:
```csharp
// WRONG: Stringly-typed access - no compile-time safety
var appName = configuration["AppSettings:AppName"]; // Returns string? - easy to typo
var retries = int.Parse(configuration["AppSettings:MaxRetries"]!); // Throws if missing

// WRONG: Section name does not match JSON key
// JSON key: "AppSettings", but record is named "AppConfig"
// Section<T> uses the type name as the section key by default
public record AppConfig { ... }
builder.Section<AppConfig>(); // Looks for "AppConfig" section, finds nothing
// Fix: Rename the record to match the JSON key, or use the overload with explicit name
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Configuration/ConfigurationOverview.html#sections

---

## CONFIG-003: Use IWritableOptions<T> for Runtime Configuration Changes

**Rule**: Inject `IWritableOptions<T>` and call `.Update()` with a record `with` expression to persist runtime changes. Use this for user preferences, feature flags, and diagnostic settings.

**Why**: `IOptions<T>` is read-only and reflects only the initial configuration snapshot. `IWritableOptions<T>` persists changes to the configuration store and notifies downstream consumers. This avoids custom file I/O for settings persistence and integrates with the Uno Platform configuration pipeline. The section does not need to exist in `appsettings.json` beforehand - it is created on first write.

**Example (C# - Settings record)**:
```csharp
public record UserPreferences(
    bool DarkMode = false,
    string Language = "en",
    bool NotificationsEnabled = true
);
```

**Example (C# - Registration)**:
```csharp
host.UseConfiguration(builder =>
    builder
        .EmbeddedSource<App>()
        .Section<UserPreferences>()
);
```

**Example (C# - Writing values at runtime)**:
```csharp
public class SettingsViewModel : ObservableObject
{
    private readonly IWritableOptions<UserPreferences> _preferences;

    public SettingsViewModel(IWritableOptions<UserPreferences> preferences)
    {
        _preferences = preferences;
    }

    public async Task ToggleDarkModeAsync()
    {
        await _preferences.Update(current =>
            current with { DarkMode = !current.DarkMode });
    }

    public async Task SetLanguageAsync(string languageCode)
    {
        await _preferences.Update(current =>
            current with { Language = languageCode });
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Using IOptions<T> and expecting writes to persist
public SettingsViewModel(IOptions<UserPreferences> preferences)
{
    // IOptions is a snapshot - mutations are lost on restart
    var prefs = preferences.Value;
    // Cannot call Update() - IOptions has no such method
}

// WRONG: Mutating the record directly instead of using "with" expression
await _preferences.Update(current =>
{
    current.DarkMode = true; // Records are immutable - this does not compile
    return current;
});
// Fix: Use "with" expression
await _preferences.Update(current => current with { DarkMode = true });
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Configuration/ConfigurationOverview.html#updating-configuration-values-at-runtime

---

## CONFIG-004: Use Environment-Specific Configuration Files

**Rule**: Create `appsettings.development.json` and `appsettings.production.json` alongside `appsettings.json`. Set the environment with `UseEnvironment()` and let `EmbeddedSource` layer them automatically.

**Why**: Environment-specific files override base settings without modifying the shared `appsettings.json`. This enables different API URLs, log levels, and feature flags per environment. The `EmbeddedSource<App>()` method loads `appsettings.<environment>.json` automatically when `includeEnvironmentSettings` is `true` (the default).

**Example (JSON - appsettings.json)**:
```json
{
  "ApiEndpoint": {
    "Url": "https://api.production.example.com",
    "UseNativeHandler": true
  },
  "Logging": {
    "MinLevel": "Warning"
  }
}
```

**Example (JSON - appsettings.development.json)**:
```json
{
  "ApiEndpoint": {
    "Url": "https://localhost:5001",
    "UseNativeHandler": false
  },
  "Logging": {
    "MinLevel": "Trace"
  }
}
```

**Example (C#)**:
```csharp
host
#if DEBUG
.UseEnvironment(Environments.Development)
#endif
.UseConfiguration(builder =>
    builder.EmbeddedSource<App>()   // Loads appsettings.json + appsettings.development.json
);
```

**Common Mistakes**:
```csharp
// WRONG: Forgetting to mark environment files as EmbeddedResource
// Only appsettings.json is embedded - environment overrides are silently ignored
<EmbeddedResource Include="appsettings.json" />
<!-- Missing: <EmbeddedResource Include="appsettings.development.json" /> -->

// WRONG: Disabling environment settings and wondering why overrides do not apply
builder.EmbeddedSource<App>(includeEnvironmentSettings: false)
// Environment-specific files are not loaded

// WRONG: Not setting UseEnvironment - defaults to Production
// appsettings.development.json is never loaded because the environment is not "Development"
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Configuration/ConfigurationOverview.html

---

## LOG-001: Wire Up Platform Logging with UseLogging

**Rule**: Call `UseLogging()` on the host builder to register platform-specific log providers (OSLog on iOS, Logcat on Android, console on Wasm/Desktop) and enable `ILogger<T>` injection.

**Why**: `UseLogging()` registers the `ILogger<T>` service and configures platform-appropriate sinks automatically. Without it, `ILogger<T>` is not available in the DI container, and constructor injection of loggers throws `InvalidOperationException`. The extension also wires up Uno internal logging (XAML parsing, layout, bindings) at the `Warning` level by default.

**Example (C#)**:
```csharp
host
.UseLogging(configure: (context, logBuilder) =>
    logBuilder
        .SetMinimumLevel(
            context.HostingEnvironment.IsDevelopment()
                ? LogLevel.Trace
                : LogLevel.Warning)
        .AddFilter("Microsoft", LogLevel.Warning)
        .AddFilter("System.Net.Http", LogLevel.Warning)
);
```

**Example (C# - Consuming ILogger)**:
```csharp
public class OrderService : IOrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public async Task PlaceOrderAsync(Order order)
    {
        _logger.LogInformation("Placing order {OrderId} for {ItemCount} items",
            order.Id, order.Items.Count);

        try
        {
            await _repository.SaveAsync(order);
            _logger.LogInformation("Order {OrderId} placed successfully", order.Id);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to place order {OrderId}", order.Id);
            throw;
        }
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Not calling UseLogging - ILogger<T> is not registered
host.ConfigureServices((ctx, services) =>
{
    services.AddSingleton<IOrderService, OrderService>();
});
// OrderService's ILogger<OrderService> constructor parameter throws at resolution

// WRONG: Using Console.WriteLine instead of ILogger
public void PlaceOrder(Order order)
{
    Console.WriteLine($"Placing order {order.Id}"); // Not captured by log providers
    // On iOS/Android, Console.WriteLine may not appear in device logs
}
```

**Uno Platform Notes**: Add `Logging` to `<UnoFeatures>` in the `.csproj`. If using `Extensions`, `Logging` is already included. For Serilog support, also add `LoggingSerilog`.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Logging/LoggingOverview.html

---

## LOG-002: Add Serilog for Structured Logging and Multiple Sinks

**Rule**: Call `UseSerilog()` on the host builder after `UseLogging()` to replace the default logging providers with Serilog's structured logging pipeline.

**Why**: Serilog captures log events as structured data (not just formatted strings), enabling rich querying in sinks like Seq, Application Insights, and Elasticsearch. It supports multiple simultaneous sinks (console + file + cloud), message templates with named properties, and sink-level filtering. The Uno Platform extension wires Serilog into the `ILogger<T>` infrastructure so all existing logging code works without changes.

**Example (XML - .csproj)**:
```xml
<UnoFeatures>
    Material;
    Extensions;
    Logging;
    LoggingSerilog;
    Toolkit;
    MVUX;
</UnoFeatures>
```

**Example (C#)**:
```csharp
host
.UseLogging(configure: (context, logBuilder) =>
    logBuilder.SetMinimumLevel(LogLevel.Debug))
.UseSerilog();
```

**Example (C# - Structured logging with Serilog message templates)**:
```csharp
public class PaymentService
{
    private readonly ILogger<PaymentService> _logger;

    public PaymentService(ILogger<PaymentService> logger)
    {
        _logger = logger;
    }

    public async Task ProcessPaymentAsync(Payment payment)
    {
        // Structured properties - captured as queryable fields, not just text
        _logger.LogInformation(
            "Processing payment {PaymentId} of {Amount:C} for customer {CustomerId}",
            payment.Id,
            payment.Amount,
            payment.CustomerId);
    }
}
// Serilog captures: { PaymentId: "pay_123", Amount: 49.99, CustomerId: "cust_456" }
// These fields are searchable in structured log sinks
```

**Common Mistakes**:
```csharp
// WRONG: String interpolation destroys structured data
_logger.LogInformation($"Processing payment {payment.Id} of {payment.Amount}");
// Serilog sees one opaque string - fields are not extractable

// CORRECT: Message template with named placeholders
_logger.LogInformation(
    "Processing payment {PaymentId} of {Amount}",
    payment.Id, payment.Amount);
// Serilog captures PaymentId and Amount as separate, queryable properties

// WRONG: Calling UseSerilog without adding LoggingSerilog to UnoFeatures
<UnoFeatures>
    Logging;          <!-- Missing LoggingSerilog -->
</UnoFeatures>
// Build error: UseSerilog extension method not found
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Logging/LoggingOverview.html#serilog

---

## LOG-003: Use Log Level Filtering to Reduce Noise

**Rule**: Configure `AddFilter()` calls in `UseLogging()` to suppress verbose output from framework categories while keeping your app's logging at a detailed level.

**Why**: Without filtering, `LogLevel.Trace` or `LogLevel.Debug` produces thousands of messages per second from `Microsoft.*`, `System.Net.Http.*`, and Uno internals, making it impossible to find your app's log entries. Category-based filters let you set `Trace` for your namespace and `Warning` for everything else, keeping logs useful during development without drowning in noise.

**Example (C#)**:
```csharp
host.UseLogging(configure: (context, logBuilder) =>
{
    if (context.HostingEnvironment.IsDevelopment())
    {
        logBuilder
            .SetMinimumLevel(LogLevel.Trace)
            // Suppress framework noise
            .AddFilter("Microsoft", LogLevel.Warning)
            .AddFilter("System", LogLevel.Warning)
            .AddFilter("Uno", LogLevel.Warning)
            // Keep detailed logging for your app
            .AddFilter("MyApp", LogLevel.Trace)
            // Targeted debug logging for specific subsystems
            .AddFilter("MyApp.Services.Payment", LogLevel.Debug);
    }
    else
    {
        logBuilder.SetMinimumLevel(LogLevel.Error);
    }
});
```

**Common Mistakes**:
```csharp
// WRONG: Setting Trace globally without filters - log output is unusable
logBuilder.SetMinimumLevel(LogLevel.Trace);
// Thousands of Uno layout, binding, and framework messages per second

// WRONG: Setting Error globally during development - missing diagnostic info
logBuilder.SetMinimumLevel(LogLevel.Error);
// You only see crashes, not the events leading up to them

// WRONG: Not using IHostEnvironment for environment-aware filtering
logBuilder.SetMinimumLevel(LogLevel.Trace); // Same verbosity in production
// Production logs fill up storage and degrade performance
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Logging/LoggingOverview.html#configuring-logging-output

---

## LOG-004: Enable Uno Internal Logging for Platform Diagnostics

**Rule**: Pass `enableUnoLogging: true` to `UseLogging()` and configure internal logging filters to diagnose XAML parsing, layout, data binding, and navigation issues.

**Why**: Uno internal logging captures events from the Uno Platform runtime itself - XAML loading failures, binding resolution errors, layout cycles, and navigation events. By default, only `Warning` and above are emitted to reduce noise. When debugging platform-level issues (e.g., a control not rendering, bindings silently failing), temporarily lowering the filter to `Debug` for specific Uno categories reveals the root cause.

**Example (C#)**:
```csharp
host.UseLogging(
    configure: (context, logBuilder) =>
    {
        logBuilder
            .SetMinimumLevel(LogLevel.Information)
            .AddFilter("MyApp", LogLevel.Debug);

        if (context.HostingEnvironment.IsDevelopment())
        {
            // Enable verbose Uno internals for specific subsystems
            logBuilder
                .AddFilter("Uno.UI.DataBinding", LogLevel.Debug)
                .AddFilter("Uno.UI.Xaml", LogLevel.Debug)
                .AddFilter("Windows.UI.Xaml.VisualStateManager", LogLevel.Debug);
        }
    },
    enableUnoLogging: true
);
```

**Common Mistakes**:
```csharp
// WRONG: Not passing enableUnoLogging - Uno internal messages are not captured
host.UseLogging(configure: (ctx, log) => log.SetMinimumLevel(LogLevel.Trace));
// Even at Trace level, Uno internal categories are not wired up

// WRONG: Setting all Uno categories to Trace in production
logBuilder.AddFilter("Uno", LogLevel.Trace);
// Massive performance impact - Uno logs thousands of layout messages per frame
// Only use for targeted debugging during development
```

**Uno Platform Notes**: Uno internal logging categories include `Uno.UI.DataBinding`, `Uno.UI.Xaml`, `Windows.UI.Xaml.Controls`, and `Windows.UI.Xaml.VisualStateManager` among others. Use targeted filters rather than a blanket `"Uno"` filter to avoid overwhelming output.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Logging/LoggingOverview.html#uno-internal-logging
