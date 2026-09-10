# Hosting and Dependency Injection

## HOSTING-001: Follow the Correct Host Builder Setup Order

**Rule**: Chain `UseConfiguration` before `UseLogging` and service-dependent extensions, and always call `.Build()` after `.Configure()`.

**Why**: Configuration must be available before logging filters or service factories that depend on `IConfiguration` execute. Misordering causes `null` configuration sections at startup, leading to silent defaults or `NullReferenceException` in factories. The host builder resolves extensions in registration order.

**Example (C#)**:
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args)
        .Configure(host =>
        {
            host
            .UseConfiguration(builder => builder
                .EmbeddedSource<App>()
                .Section<AppSettings>()
            )
            .UseLogging(configure: (context, logBuilder) =>
                logBuilder.SetMinimumLevel(
                    context.HostingEnvironment.IsDevelopment()
                        ? LogLevel.Trace
                        : LogLevel.Warning)
            )
            .ConfigureServices((context, services) =>
            {
                services.AddSingleton<IWeatherService, WeatherService>();
            });
        });

    Host = appBuilder.Build();
    MainWindow = appBuilder.Window;
    MainWindow.Activate();
}
```

**Common Mistakes**:
```csharp
// WRONG: ConfigureServices before UseConfiguration - IConfiguration not yet populated
host.ConfigureServices((ctx, services) =>
{
    var url = ctx.Configuration["ApiUrl"]; // null - configuration not loaded yet
    services.AddSingleton(new ApiClient(url!));
})
.UseConfiguration(builder => builder.EmbeddedSource<App>());
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Walkthrough/HostingSetup.howto.html

---

## HOSTING-002: Use TryAdd for Default Service Registrations

**Rule**: Use `TryAddSingleton`, `TryAddTransient`, or `TryAddScoped` when registering default implementations that consumers may override.

**Why**: `TryAdd*` methods skip registration if the service type is already registered, preventing accidental replacement of custom implementations. Standard `Add*` methods always append, which means the last registration wins when resolving a single instance. Libraries and shared modules should use `TryAdd*`; app-level code can use `Add*` when it must guarantee its implementation.

**Example (C#)**:
```csharp
// In a shared service library - allows app to override
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddWeatherDefaults(this IServiceCollection services)
    {
        services.TryAddSingleton<IWeatherService, OpenWeatherService>();
        services.TryAddSingleton<ICacheService, InMemoryCacheService>();
        return services;
    }
}

// In App.xaml.cs - explicit override takes precedence
host.ConfigureServices((context, services) =>
{
    // This registration wins because it comes first
    services.AddSingleton<IWeatherService, CustomWeatherService>();

    // TryAdd inside AddWeatherDefaults will skip IWeatherService (already registered)
    services.AddWeatherDefaults();
});
```

**Common Mistakes**:
```csharp
// WRONG: AddSingleton always appends - last one wins for single resolution
services.AddSingleton<IWeatherService, OpenWeatherService>();
services.AddSingleton<IWeatherService, CustomWeatherService>(); // This one resolves

// WRONG: TryAdd at app level when you need to guarantee your implementation
services.TryAddSingleton<IWeatherService, CustomWeatherService>();
// If a library already registered IWeatherService, your custom one is silently skipped
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/DependencyInjection/DependencyInjectionOverview.html

---

## HOSTING-003: Prefer Constructor Injection Over Service Locator

**Rule**: Resolve services through constructor parameters on ViewModels and services. Reserve `Host.Services.GetService<T>()` for rare root-level bootstrapping scenarios.

**Why**: Constructor injection makes dependencies explicit, testable, and verifiable at compile time. The service locator pattern (`Host.Services.GetService<T>()`) hides dependencies, makes unit testing harder, and couples code to the DI container. The Uno Platform host automatically injects constructor parameters for ViewModels registered via navigation or MVUX.

**Example (C#)**:
```csharp
// CORRECT: Constructor injection - dependencies are explicit
public class MainViewModel : ObservableObject
{
    private readonly IWeatherService _weatherService;
    private readonly ILogger<MainViewModel> _logger;

    public MainViewModel(
        IWeatherService weatherService,
        ILogger<MainViewModel> logger)
    {
        _weatherService = weatherService;
        _logger = logger;
    }

    public async Task LoadForecastAsync()
    {
        _logger.LogInformation("Loading forecast");
        var forecast = await _weatherService.GetForecastAsync();
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Service locator - hides dependencies, untestable
public class MainViewModel : ObservableObject
{
    public async Task LoadForecastAsync()
    {
        // Hidden dependency - not visible in constructor
        var service = App.Instance.Host.Services.GetRequiredService<IWeatherService>();
        var forecast = await service.GetForecastAsync();
    }
}
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/DependencyInjection/DependencyInjectionOverview.html#resolving-services

---

## HOSTING-004: Register Named Services for Multi-Implementation Scenarios

**Rule**: Use `AddNamedSingleton` / `AddNamedTransient` and resolve with `GetRequiredNamedService<T>("name")` when you need multiple implementations of the same interface differentiated at runtime.

**Why**: Standard DI only resolves a single implementation per interface (the last registered). Named services let you register multiple implementations with distinct keys and resolve the correct one by name. This avoids the need for factory patterns or wrapper classes when the difference is purely configurational.

**Example (C#)**:
```csharp
// Registration
host.ConfigureServices((context, services) =>
{
    services.AddNamedSingleton<IStorageService, LocalStorageService>("local");
    services.AddNamedSingleton<IStorageService, CloudStorageService>("cloud");
});

// Resolution - in a service or view model
public class SyncManager
{
    private readonly IStorageService _local;
    private readonly IStorageService _cloud;

    public SyncManager(IServiceProvider serviceProvider)
    {
        _local = serviceProvider.GetRequiredNamedService<IStorageService>("local");
        _cloud = serviceProvider.GetRequiredNamedService<IStorageService>("cloud");
    }

    public async Task SyncAsync()
    {
        var data = await _local.LoadAsync();
        await _cloud.SaveAsync(data);
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Registering two implementations without names - only the last one resolves
services.AddSingleton<IStorageService, LocalStorageService>();
services.AddSingleton<IStorageService, CloudStorageService>();
// GetService<IStorageService>() returns CloudStorageService only

// WRONG: Using GetNamedService (nullable) without checking for null
var svc = provider.GetNamedService<IStorageService>("typo"); // Returns null silently
svc.LoadAsync(); // NullReferenceException
// Use GetRequiredNamedService instead - it throws InvalidOperationException with a clear message
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/DependencyInjection/DependencyInjectionOverview.html#named-services

---

## HOSTING-005: Use IHostEnvironment to Vary Behavior by Environment

**Rule**: Set the hosting environment with `UseEnvironment()` and query it via `IHostEnvironment.IsDevelopment()` in service registration and logging configuration.

**Why**: Environment-aware configuration lets you enable verbose logging, debug handlers, and mock services during development without shipping that overhead in production. The `HostBuilderContext` parameter in `ConfigureServices` and `UseLogging` exposes `HostingEnvironment` for branching.

**Example (C#)**:
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args)
        .Configure(host =>
        {
            host
#if DEBUG
            .UseEnvironment(Environments.Development)
#endif
            .UseLogging(configure: (context, logBuilder) =>
                logBuilder.SetMinimumLevel(
                    context.HostingEnvironment.IsDevelopment()
                        ? LogLevel.Trace
                        : LogLevel.Error)
            )
            .ConfigureServices((context, services) =>
            {
                if (context.HostingEnvironment.IsDevelopment())
                {
                    services.AddSingleton<IAnalyticsService, NoOpAnalyticsService>();
                    services.AddTransient<DelegatingHandler, DebugHttpHandler>();
                }
                else
                {
                    services.AddSingleton<IAnalyticsService, AppInsightsService>();
                }
            });
        });

    Host = appBuilder.Build();
}
```

**Common Mistakes**:
```csharp
// WRONG: Using #if DEBUG everywhere instead of IHostEnvironment
services.AddSingleton<IAnalyticsService>(
#if DEBUG
    new NoOpAnalyticsService()    // Cannot be changed without recompiling
#else
    new AppInsightsService()
#endif
);

// WRONG: Forgetting to set UseEnvironment - IsDevelopment() returns false even in DEBUG
host.UseLogging(configure: (ctx, log) =>
    log.SetMinimumLevel(
        ctx.HostingEnvironment.IsDevelopment() // Always false without UseEnvironment
            ? LogLevel.Trace
            : LogLevel.Error)
);
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Logging/LoggingOverview.html

---

## HOSTING-006: Use Service Implementation Factories for Complex Construction

**Rule**: Use the factory overload of `AddSingleton<TService>(Func<IServiceProvider, TService>)` when a service requires runtime configuration or dependencies that are not directly injectable.

**Why**: Factory methods let you resolve other services, read configuration values, and perform conditional construction logic. This is essential when an implementation needs initialization steps beyond what constructor injection provides, or when the concrete type depends on configuration.

**Example (C#)**:
```csharp
host.ConfigureServices((context, services) =>
{
    services.AddSingleton<IApiClient>(serviceProvider =>
    {
        var configuration = serviceProvider.GetRequiredService<IConfiguration>();
        var logger = serviceProvider.GetRequiredService<ILogger<ApiClient>>();
        var baseUrl = configuration["ApiSettings:BaseUrl"]
            ?? throw new InvalidOperationException("ApiSettings:BaseUrl is required");

        var client = new ApiClient(baseUrl, logger);
        client.SetTimeout(TimeSpan.FromSeconds(30));
        return client;
    });
});
```

**Common Mistakes**:
```csharp
// WRONG: Constructing service inline without access to DI container
services.AddSingleton<IApiClient>(new ApiClient("https://api.example.com", null!));
// The logger is null - service misses its dependency

// WRONG: Resolving services inside a constructor of a manually-newed service
var client = new ApiClient(
    Host.Services.GetRequiredService<IConfiguration>()["ApiUrl"]!,
    Host.Services.GetRequiredService<ILogger<ApiClient>>()
);
// Host may not be built yet at registration time
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/DependencyInjection/DependencyInjectionOverview.html#service-implementation-factories

---

## HOSTING-007: Store the IHost Reference for Root-Level Service Access

**Rule**: Assign the built host to a property on `App` so root-level code that cannot use constructor injection can resolve services.

**Why**: Some platform callbacks (e.g., `OnActivated`, deep link handlers) execute outside the DI-managed object graph. Storing the `IHost` reference provides a controlled escape hatch. However, all ViewModel and service classes should still use constructor injection.

**Example (C#)**:
```csharp
public sealed partial class App : Application
{
    public IHost Host { get; private set; } = default!;

    protected override void OnLaunched(LaunchActivatedEventArgs args)
    {
        var appBuilder = this.CreateBuilder(args)
            .Configure(host =>
            {
                host.ConfigureServices((context, services) =>
                {
                    services.AddSingleton<IDeepLinkService, DeepLinkService>();
                });
            });

        var app = appBuilder.Build();
        Host = app.Host;
        MainWindow = app.Window;
        MainWindow.Activate();
    }

    // Platform callback - cannot use constructor injection
    protected override void OnActivated(IActivatedEventArgs args)
    {
        if (args is IProtocolActivatedEventArgs protocolArgs)
        {
            var deepLinkService = Host.Services.GetRequiredService<IDeepLinkService>();
            deepLinkService.HandleUri(protocolArgs.Uri);
        }
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Accessing Host before Build() completes
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var appBuilder = this.CreateBuilder(args).Configure(host => { });
    var service = Host.Services.GetService<IMyService>(); // Host is still default/null!
    Host = appBuilder.Build().Host;
}
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Walkthrough/HostingSetup.howto.html
