# Platform-Specific Code (High Priority)

## Conditional Compilation Directives

**When to apply:** Platform-specific implementations

### Platform Defines
```csharp
#if __ANDROID__
    // Android-specific code
    using Android.App;
#elif __IOS__
    // iOS-specific code
    using UIKit;
#elif __WINDOWS__
    // Windows-specific code
    using Windows.UI.Core;
#elif __SKIA__
    // Skia platforms (Linux, macOS via Skia, etc.)
#elif __WASM__
    // WebAssembly-specific code
#endif
```

### Runtime Platform Detection
```csharp
#if NET6_0_OR_GREATER
    if (OperatingSystem.IsBrowser())
    {
        // WebAssembly-specific logic
    }
    else if (OperatingSystem.IsAndroid())
    {
        // Android runtime logic
    }
    else if (OperatingSystem.IsIOS())
    {
        // iOS runtime logic
    }
#endif
```

### References
- Source: C:\Users\Platform006\uno.extensions\src\Uno.Extensions.Logging

## Platforms Folder Structure

**When to apply:** Single Project with platform-specific code

### Folder Organization
```
MyApp/
├── Platforms/
│   ├── Android/
│   │   ├── MainActivity.Android.cs
│   │   ├── Resources/
│   │   └── AndroidManifest.xml
│   ├── iOS/
│   │   ├── Main.iOS.cs
│   │   ├── Info.plist
│   │   └── Entitlements.plist
│   ├── Desktop/
│   │   └── Program.cs
│   └── WebAssembly/
│       └── Program.cs
```

### Platform Entry Points

**Android (MainActivity.Android.cs):**
```csharp
namespace MyApp;

[Activity(
    MainLauncher = true,
    ConfigurationChanges = global::Uno.UI.ActivityHelper.AllConfigChanges,
    WindowSoftInputMode = SoftInput.AdjustPan | SoftInput.StateHidden
)]
public class MainActivity : Microsoft.UI.Xaml.ApplicationActivity
{
}
```

**iOS (Main.iOS.cs):**
```csharp
namespace MyApp;

public class Program
{
    static void Main(string[] args)
    {
        Microsoft.UI.Xaml.Application.Start(_ => new App());
    }
}
```

**Desktop (Program.cs):**
```csharp
namespace MyApp;

public class Program
{
    [STAThread]
    static void Main(string[] args)
    {
        Microsoft.UI.Xaml.Application.Start(_ => new App());
    }
}
```

**WebAssembly (Program.cs):**
```csharp
namespace MyApp;

public class Program
{
    private static App? _app;

    static async Task Main(string[] args)
    {
        Microsoft.UI.Xaml.Application.Start(_ => _app = new App());
        await Task.Yield();
    }
}
```

### References
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\Platforms

## Platform-Specific Resource Access

**When to apply:** Assets, icons, splash screens

### Android Resources
```
Platforms/Android/Resources/
├── drawable/
│   └── icon.png
├── drawable-hdpi/
├── drawable-xhdpi/
├── drawable-xxhdpi/
├── drawable-xxxhdpi/
├── mipmap-anydpi-v26/
└── values/
    └── Strings.xml
```

### iOS Resources
```
Platforms/iOS/Resources/
├── LaunchScreen.storyboard
├── Assets.xcassets/
│   ├── AppIcon.appiconset/
│   └── LaunchImage.launchimage/
```

### WebAssembly Assets
```
Platforms/WebAssembly/
└── wwwroot/
    └── favicon.ico
```

### Shared Assets
```
Assets/
├── Icons/
├── Images/
├── Fonts/
└── Lottie/
```

### References
- Official docs: Uno Platform asset management

## Platform-Specific Service Implementation

**When to apply:** Services with platform-specific implementations

### Interface in Shared Code
```csharp
public interface IDeviceInfoService
{
    string GetDeviceModel();
    string GetOSVersion();
    string GetAppVersion();
}
```

### Partial Class Pattern
```csharp
// Shared/DeviceInfoService.cs
public partial class DeviceInfoService : IDeviceInfoService
{
    public string GetDeviceModel() => GetDeviceModelPlatform();
    public string GetOSVersion() => GetOSVersionPlatform();
    public string GetAppVersion() => GetAppVersionPlatform();

    partial string GetDeviceModelPlatform();
    partial string GetOSVersionPlatform();
    partial string GetAppVersionPlatform();
}

// Platforms/Android/DeviceInfoService.Android.cs
public partial class DeviceInfoService
{
    partial string GetDeviceModelPlatform()
    {
        return $"{Android.OS.Build.Manufacturer} {Android.OS.Build.Model}";
    }

    partial string GetOSVersionPlatform()
    {
        return $"Android {Android.OS.Build.VERSION.Release}";
    }

    partial string GetAppVersionPlatform()
    {
        var context = Android.App.Application.Context;
        var packageInfo = context.PackageManager?.GetPackageInfo(context.PackageName!, 0);
        return packageInfo?.VersionName ?? "Unknown";
    }
}

// Platforms/iOS/DeviceInfoService.iOS.cs
public partial class DeviceInfoService
{
    partial string GetDeviceModelPlatform()
    {
        return UIDevice.CurrentDevice.Model;
    }

    partial string GetOSVersionPlatform()
    {
        return $"iOS {UIDevice.CurrentDevice.SystemVersion}";
    }

    partial string GetAppVersionPlatform()
    {
        return NSBundle.MainBundle.InfoDictionary?["CFBundleShortVersionString"]?.ToString() ?? "Unknown";
    }
}
```

### DI Registration
```csharp
services.AddSingleton<IDeviceInfoService, DeviceInfoService>();
```

### References
- Pattern from uno.extensions Logging module

## HTTP Handler Configuration

**When to apply:** Platform-specific HTTP configuration

### Pattern
```csharp
public static IHttpClientBuilder ConfigurePrimaryHttpMessageHandler(
    this IHttpClientBuilder httpClientBuilder,
    bool useNativeHandler = true)
{
    return httpClientBuilder.ConfigurePrimaryAndInnerHttpMessageHandler<HttpMessageHandler>(useNativeHandler);
}

internal static IHttpClientBuilder ConfigurePrimaryAndInnerHttpMessageHandler<THandler>(
    this IHttpClientBuilder builder,
    bool useNativeHandler = true)
    where THandler : HttpMessageHandler
{
#if __IOS__ || __ANDROID__
    if (useNativeHandler)
    {
        builder.ConfigurePrimaryHttpMessageHandler(() => new Xamarin.Android.Net.AndroidMessageHandler());
    }
#elif __WINDOWS__
    if (useNativeHandler)
    {
        builder.ConfigurePrimaryHttpMessageHandler(() => new System.Net.Http.WinHttpHandler());
    }
#endif

    return builder;
}
```

### Benefits
- Uses native HTTP stacks for better performance
- Respects system proxy settings
- Better certificate validation

### References
- Source: C:\Users\Platform006\uno.extensions\src\Uno.Extensions.Http

## WebAssembly-Specific Considerations

**When to apply:** WASM projects

### Async Main Pattern
```csharp
static async Task Main(string[] args)
{
    Microsoft.UI.Xaml.Application.Start(_ => _app = new App());
    await Task.Yield(); // Required for WASM
}
```

### WASM Configuration
```xml
<PropertyGroup Condition="'$(TargetFramework)' == 'net9.0-browserwasm'">
    <WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
</PropertyGroup>
```

### References
- Official WASM bootstrap docs
