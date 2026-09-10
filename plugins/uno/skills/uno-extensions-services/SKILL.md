---
name: uno-extensions-services
description: "Uno Platform Extensions for hosting, dependency injection, authentication, HTTP clients, configuration, logging, and storage. Use when: (1) Setting up IHostBuilder and DI container, (2) Adding MSAL or OIDC authentication, (3) Configuring HTTP clients with Kiota or Refit, (4) Loading appsettings.json configuration with IOptions, (5) Setting up logging with ILogger or Serilog, (6) Using IWriteableOptions for runtime config changes. Do NOT use for: project setup or MVVM patterns (see uno-platform-agent) or UI controls (see uno-toolkit)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation)"
metadata:
  version: "1.0.0"
---

# Uno Platform Extensions Services

Patterns for the Uno.Extensions service layer: hosting, DI, authentication, HTTP, configuration, logging, and storage.

## Critical Rules

### Hosting and Dependency Injection

- **Always start with `.UseConfiguration()` before other extensions** - Configuration is a dependency of most other extensions
- **Use `TryAddSingleton` / `TryAddTransient`** - Allows downstream code to override registrations
- **Constructor injection is the primary resolution pattern** - Inject `IServiceProvider` only as a last resort
- **Named services** use `AddNamedSingleton<TService, TImpl>("name")` and resolve with `GetRequiredNamedService<TService>("name")`

```csharp
// App.xaml.cs - host builder setup
var builder = this.CreateBuilder(args)
    .Configure(host => host
        .UseConfiguration()
        .UseLogging()
        .UseSerilog()
        .UseAuthentication()
        .UseHttp()
        .ConfigureServices((ctx, services) =>
        {
            services.TryAddSingleton<IMyService, MyService>();
        })
    );
```

### UnoFeatures Required

```xml
<UnoFeatures>
    Hosting;
    Configuration;
    Logging;
    LoggingSerilog;
    Http;
    HttpKiota;      <!-- or HttpRefit -->
    Authentication;
    AuthenticationMsal; <!-- or AuthenticationOidc -->
    Storage;
</UnoFeatures>
```

## Authentication

- **MSAL** for Microsoft identity (Azure AD, Azure AD B2C)
- **OIDC** for any OpenID Connect provider (IdentityServer, Auth0, Okta)
- Add `AuthenticationMsal` or `AuthenticationOidc` to UnoFeatures
- MSAL 5.2+: `AddMsal()` requires a `Window` parameter

```csharp
.UseAuthentication(auth =>
    auth.AddMsal(msal =>
        msal
            .Builder(b => b.WithAuthority("https://login.microsoftonline.com/{tenantId}"))
            .Scopes(new[] { "User.Read" })
    )
)
```

```csharp
// OIDC
.UseAuthentication(auth =>
    auth.AddOidc(oidc =>
        oidc
            .Authority("https://your-identity-server.com")
            .ClientId("your-client-id")
            .Scopes(new[] { "openid", "profile" })
    )
)
```

## HTTP Clients

- **Kiota (preferred)**: Generate strongly-typed clients from OpenAPI specs
- **Refit**: Interface-based HTTP clients with attribute routing
- Configure endpoints in `appsettings.json`, not hardcoded URLs

```csharp
// Kiota setup
.UseHttp(http =>
    http.AddClient<MyApiClient>(ctx =>
        new MyApiClient(
            new HttpClientRequestAdapter(
                new AnonymousAuthenticationProvider(),
                httpClient: ctx.HttpClient)))
)
```

```csharp
// Refit setup
.UseHttp(http =>
    http.AddRefitClient<IMyApi>(ctx =>
        new RefitSettings())
)
```

```json
// appsettings.json
{
  "MyApi": {
    "Url": "https://api.example.com",
    "UseNativeHandler": true
  }
}
```

## Configuration

- Embed `appsettings.json` as `EmbeddedResource` in the project
- Use `IOptions<T>` for read-only, `IWriteableOptions<T>` for runtime changes
- Environment-specific overrides: `appsettings.development.json`

```csharp
.UseConfiguration(config =>
    config
        .EmbeddedSource<App>()
        .Section<MySettings>()
)
```

```csharp
// Consuming in a ViewModel
public class MyViewModel
{
    public MyViewModel(IOptions<MySettings> settings)
    {
        var baseUrl = settings.Value.BaseUrl;
    }
}
```

## Logging

- `UseLogging()` registers `ILogger<T>` in DI
- `UseSerilog()` adds Serilog with structured logging and configurable sinks
- Add `LoggingSerilog` to UnoFeatures for Serilog support

```csharp
.UseLogging(configure: logBuilder =>
    logBuilder
        .SetMinimumLevel(LogLevel.Information)
        .AddFilter("Uno", LogLevel.Warning)
)
.UseSerilog()
```

## Common Mistakes

- Forgetting the `Window` parameter for `AddMsal()` on Uno.Extensions 5.2+
- Not registering redirect URIs per platform (Android needs intent filter, iOS needs URL scheme)
- Using `Authentication` UnoFeature alone without `AuthenticationMsal` or `AuthenticationOidc`
- Hardcoding base URLs instead of using `appsettings.json` configuration
- Not using `UseNativeHandler: true` for platform-optimized networking
- Creating `HttpClient` instances manually instead of through DI
- Calling `.UseHttp()` or `.UseAuthentication()` before `.UseConfiguration()` (configuration must come first)
- Injecting `IServiceProvider` directly instead of using constructor injection

## Related Skills

| Skill | Use instead when... |
|-------|-------------------|
| `uno-platform-agent` | Setting up project structure, MVVM/MVUX, navigation, or build config |
| `uno-build-troubleshoot` | Upgrading Uno.Extensions versions or fixing breaking changes |
| `winui-xaml` | XAML layout, binding, async, or memory management best practices |
| `uno-wasm-pwa` | WebAssembly-specific hosting, debugging, or deployment |

## Detailed References

- [references/01-hosting-di.md](references/01-hosting-di.md) - Read when setting up host builder, registering services, or using named services
- [references/02-authentication.md](references/02-authentication.md) - Read when adding MSAL, OIDC, or custom auth providers
- [references/03-http-clients.md](references/03-http-clients.md) - Read when configuring Kiota, Refit, or custom HTTP handlers
- [references/04-configuration-logging.md](references/04-configuration-logging.md) - Read when setting up appsettings, IOptions, ILogger, or Serilog
