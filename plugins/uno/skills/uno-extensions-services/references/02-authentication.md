# Authentication

## AUTH-001: Pass the Window Instance When Configuring MSAL (5.2+)

**Rule**: Use the `Configure((host, window) => ...)` overload and pass `window` to `AddMsal(window)` in Uno.Extensions 5.2 and later.

**Why**: Starting with Uno.Extensions 5.2, `MsalAuthenticationProvider` requires a `Window` instance to display the authentication dialog. Without it, MSAL throws `MsalClientException` with the message "Only loopback redirect uri is supported" on desktop platforms, and the auth flow silently fails on mobile. The `Configure` overload that provides `(host, window)` is the only way to access the initialized window during host setup.

**Example (C#)**:
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var builder = this.CreateBuilder(args)
        // Use the overload that provides the Window parameter
        .Configure((host, window) =>
        {
            host
            .UseAuthentication(auth =>
            {
                auth.AddMsal(window, msal =>
                    msal
                    .Builder(msalBuilder =>
                        msalBuilder.WithClientId("161a9fb5-3b16-487a-81a2-ac45dcc0ad3b"))
                    .Scopes(new[] { "User.Read", "Tasks.ReadWrite" })
                );
            });
        });

    Host = builder.Build();
}
```

**Common Mistakes**:
```csharp
// WRONG (5.2+): Using Configure(host => ...) without Window - causes MsalClientException
.Configure(host =>
{
    host.UseAuthentication(auth =>
    {
        auth.AddMsal(); // Missing Window parameter - throws at runtime
    });
})

// WRONG: Passing null for window
auth.AddMsal(null!, msal => msal.Builder(...));
// Runtime crash: NullReferenceException when MSAL tries to display the login dialog
```

**Uno Platform Notes**: This breaking change was introduced in Uno.Extensions 5.2. Projects upgrading from earlier versions must switch from `Configure(host => ...)` to `Configure((host, window) => ...)` and add the `window` argument to `AddMsal()`.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/UpdatingExtensions.html#msal-authentication

---

## AUTH-002: Configure MSAL via appsettings.json for Clean Separation

**Rule**: Place MSAL `ClientId` and `Scopes` in the `"Msal"` section of `appsettings.json` rather than hardcoding them in `App.xaml.cs`.

**Why**: Separating credentials from code enables environment-specific overrides (`appsettings.development.json` vs `appsettings.production.json`), reduces merge conflicts, and makes it easy to rotate client IDs without recompiling. The `AddMsal()` method automatically reads from the `"Msal"` configuration section when no inline builder is provided.

**Example (JSON - appsettings.json)**:
```json
{
  "Msal": {
    "ClientId": "161a9fb5-3b16-487a-81a2-ac45dcc0ad3b",
    "Scopes": [ "User.Read", "Tasks.Read", "Tasks.ReadWrite" ]
  }
}
```

**Example (C#)**:
```csharp
.Configure((host, window) =>
{
    host
    .UseConfiguration(builder => builder.EmbeddedSource<App>())
    .UseAuthentication(auth =>
    {
        // Reads ClientId and Scopes from "Msal" section automatically
        auth.AddMsal(window);
    });
})
```

**Common Mistakes**:
```csharp
// WRONG: Hardcoding credentials in source code
auth.AddMsal(window, msal =>
    msal.Builder(b => b.WithClientId("161a9fb5-..."))
        .Scopes(new[] { "User.Read" })
);
// Credentials are compiled into the binary and visible in source control

// WRONG: Forgetting UseConfiguration - MSAL cannot read appsettings.json
host.UseAuthentication(auth => auth.AddMsal(window));
// Throws: "No ClientId was specified" because configuration was never loaded
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Authentication/HowTo-MsalAuthentication.html#3-configure-the-provider

---

## AUTH-003: Set Up OIDC with Authority, ClientId, and Scopes

**Rule**: Configure `OidcAuthenticationProvider` with `Authority`, `ClientId`, `ClientSecret` (if confidential), `Scope`, and `RedirectUri` via the `"Oidc"` configuration section.

**Why**: OIDC requires an authority URL for discovery, a client identifier for the relying party, and scopes to determine the claims and tokens returned. Missing any of these causes the authentication flow to fail silently or return incomplete tokens. The `offline_access` scope is required for refresh token support.

**Example (JSON - appsettings.json)**:
```json
{
  "Oidc": {
    "Authority": "https://demo.duendesoftware.com/",
    "ClientId": "interactive.confidential",
    "ClientSecret": "secret",
    "Scope": "openid profile email api offline_access",
    "RedirectUri": "oidc-auth://callback"
  }
}
```

**Example (C#)**:
```csharp
.Configure(host =>
{
    host
    .UseConfiguration(builder => builder.EmbeddedSource<App>())
    .UseAuthentication(auth =>
    {
        auth.AddOidc();
    });
})
```

**Common Mistakes**:
```csharp
// WRONG: Omitting "openid" scope - required for OIDC compliance
{
  "Oidc": {
    "Authority": "https://idp.example.com/",
    "ClientId": "my-app",
    "Scope": "profile email"  // Missing "openid" - identity token will be absent
  }
}

// WRONG: Omitting "offline_access" and expecting token refresh to work
// Without offline_access, no refresh token is issued and the session expires silently
```

**Uno Platform Notes**: On WebAssembly, the redirect URI is automatically discovered via `WebAuthenticationBroker.GetCurrentApplicationCallbackUri()`. This is ON by default for Wasm but opt-in on other platforms. Use `.AutoRedirectUriFromAuthenticationBroker()` to enable it explicitly.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Authentication/HowTo-OidcAuthentication.html

---

## AUTH-004: Configure Per-Platform Redirect URIs

**Rule**: Register platform-specific redirect URIs for Android (intent filter), iOS/macOS (URL scheme in Info.plist), and WebAssembly (automatic via broker or explicit origin).

**Why**: Each platform has a different mechanism for receiving the OAuth callback after authentication completes. Misconfigured redirect URIs cause the auth flow to hang (the browser redirects but the app never receives the callback) or throw "redirect_uri mismatch" errors from the identity provider. The redirect URI registered in code must exactly match what is registered in the identity provider's app registration.

**Example (Android - AndroidManifest.xml intent filter for MSAL)**:
```xml
<activity android:name="microsoft.identity.client.BrowserTabActivity"
          android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="msal161a9fb5-3b16-487a-81a2-ac45dcc0ad3b"
              android:host="auth" />
    </intent-filter>
</activity>
```

**Example (iOS/macOS - Info.plist URL scheme for OIDC)**:
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>Authentication Callback</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>oidc-auth</string>
        </array>
    </dict>
</array>
```

**Example (WebAssembly - automatic)**:
```csharp
// Wasm uses WebAuthenticationBroker by default - no explicit redirect URI needed
// The broker auto-discovers the origin URL (e.g., https://myapp.com/authentication/login-callback)
auth.AddOidc();
// Or explicitly set for non-Wasm platforms:
auth.AddOidc()
    .AutoRedirectUriFromAuthenticationBroker();
```

**Common Mistakes**:
```xml
<!-- WRONG (Android): Scheme does not match ClientId format for MSAL -->
<data android:scheme="myapp" android:host="auth" />
<!-- MSAL requires the scheme to be "msal{ClientId}" -->

<!-- WRONG (iOS): Missing URL scheme entirely - browser redirects but app does not receive -->
<!-- No CFBundleURLTypes entry in Info.plist -->
```

**Reference**: https://platform.uno/docs/articles/interop/MSAL.html#android

---

## AUTH-005: Use IAuthenticationService for Login, Logout, and Token Access

**Rule**: Inject `IAuthenticationService` into ViewModels to perform login, logout, and check authentication state. Do not interact with MSAL or OIDC libraries directly.

**Why**: `IAuthenticationService` provides a unified abstraction across all authentication providers. It handles token storage, automatic refresh, and credential management. Bypassing it to call MSAL or OIDC libraries directly skips token caching, creates duplicate auth sessions, and breaks the extensions' lifecycle management.

**Example (C#)**:
```csharp
public class LoginViewModel : ObservableObject
{
    private readonly IAuthenticationService _authService;
    private readonly ILogger<LoginViewModel> _logger;

    public LoginViewModel(
        IAuthenticationService authService,
        ILogger<LoginViewModel> logger)
    {
        _authService = authService;
        _logger = logger;
        LoginCommand = new AsyncRelayCommand(LoginAsync);
        LogoutCommand = new AsyncRelayCommand(LogoutAsync);
    }

    public ICommand LoginCommand { get; }
    public ICommand LogoutCommand { get; }

    private async Task LoginAsync()
    {
        var success = await _authService.LoginAsync(
            new Dictionary<string, string>());

        if (success)
        {
            _logger.LogInformation("User authenticated successfully");
        }
        else
        {
            _logger.LogWarning("Authentication failed or was cancelled");
        }
    }

    private async Task LogoutAsync()
    {
        await _authService.LogoutAsync(CancellationToken.None);
        _logger.LogInformation("User logged out");
    }
}
```

**Common Mistakes**:
```csharp
// WRONG: Calling MSAL directly bypasses token caching and refresh
var app = PublicClientApplicationBuilder.Create(clientId).Build();
var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
// Tokens are not stored in credential storage - next launch requires fresh login

// WRONG: Not awaiting LoginAsync - fire-and-forget loses exceptions
LoginCommand = new RelayCommand(() => _authService.LoginAsync(credentials));
// Use AsyncRelayCommand so exceptions propagate correctly
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Authentication/HowTo-MsalAuthentication.html#4-use-the-provider-in-your-application

---

## AUTH-006: Add UnoFeatures for the Chosen Authentication Provider

**Rule**: Add `AuthenticationMsal` or `AuthenticationOidc` to `<UnoFeatures>` in the class library `.csproj` file. Do not add raw NuGet references for the auth extensions.

**Why**: The `UnoFeatures` system in the Uno SDK manages transitive dependencies, version alignment, and platform-specific packages automatically. Adding `AuthenticationMsal` pulls in MSAL, Uno.Extensions.Authentication, and platform bridges. Manually adding NuGet references risks version mismatches between MSAL and the Uno authentication wrapper, causing `TypeLoadException` or `MissingMethodException` at runtime.

**Example (XML - .csproj)**:
```xml
<!-- For MSAL (Azure AD / Azure AD B2C) -->
<UnoFeatures>
    Material;
    Extensions;
    AuthenticationMsal;
    Toolkit;
    MVUX;
</UnoFeatures>

<!-- For OIDC (IdentityServer, Auth0, Okta, etc.) -->
<UnoFeatures>
    Material;
    Extensions;
    AuthenticationOidc;
    Toolkit;
    MVUX;
</UnoFeatures>
```

**Common Mistakes**:
```xml
<!-- WRONG: Adding Authentication without specifying provider -->
<UnoFeatures>
    Authentication;
</UnoFeatures>
<!-- "Authentication" adds the base extension only - no MSAL or OIDC provider -->

<!-- WRONG: Manual NuGet references instead of UnoFeatures -->
<PackageReference Include="Uno.Extensions.Authentication.MSAL.WinUI" Version="4.1.0" />
<PackageReference Include="Microsoft.Identity.Client" Version="4.58.0" />
<!-- Version mismatch risk - let UnoFeatures manage alignment -->
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Authentication/AuthenticationOverview.html#installation

---

## AUTH-007: Use ConfigureOidcClientOptions for Advanced OIDC Settings

**Rule**: Use the `.ConfigureOidcClientOptions()` extension method to access the underlying `OidcClientOptions` from `IdentityModel.OidcClient` for advanced protocol configuration.

**Why**: The default OIDC configuration covers standard scenarios, but some identity providers require non-default settings such as disabling Pushed Authorization Requests (PAR), custom discovery policies, or token validation overrides. `ConfigureOidcClientOptions` provides direct access without forking or wrapping the provider.

**Example (C#)**:
```csharp
host.UseAuthentication(auth =>
{
    auth.AddOidc()
        .ConfigureOidcClientOptions(options =>
        {
            // Disable PAR if the identity provider does not support it
            options.DisablePushedAuthorization = true;

            // Require identity token on refresh (stricter validation)
            options.Policy.RequireIdentityTokenOnRefreshTokenResponse = true;

            // Relax issuer validation for dev environments
            options.Policy.Discovery.ValidateIssuerName = false;
        });
});
```

**Common Mistakes**:
```csharp
// WRONG: Trying to set advanced options via appsettings.json
// OidcClientOptions are not bindable from configuration - use ConfigureOidcClientOptions

// WRONG: Disabling issuer validation in production
options.Policy.Discovery.ValidateIssuerName = false;
// Only acceptable during development against local identity servers
// In production this opens the door to token substitution attacks
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Authentication/HowTo-OidcAuthentication.html
