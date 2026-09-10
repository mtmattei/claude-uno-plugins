# Common Errors Reference

Detailed patterns for diagnosing and resolving the most frequently encountered build errors, runtime failures, and IDE issues in Uno Platform development.

---

### Fix UNOB0011 and UNOB0013: TFM Ordering Errors

**Rule**: When you encounter UNOB0011 ("Project is not a valid Uno Platform project") or UNOB0013, immediately check the `<TargetFrameworks>` element and move any bare `net9.0` entry to the end of the list.

**Why**: The Uno.Sdk inspects the first TFM to determine the project type. When `net9.0` (the base class library TFM) appears first, the SDK concludes the project is a standard .NET library rather than an Uno Platform app, and disables all Uno Platform build targets. This produces UNOB0011/UNOB0013 at build time and causes VS2022 IntelliSense to report false errors on every Uno Platform API reference.

**Example (csproj - broken)**:
```xml
<!-- CAUSES UNOB0011/UNOB0013 -->
<PropertyGroup>
    <TargetFrameworks>net9.0;net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop</TargetFrameworks>
</PropertyGroup>
```

**Example (csproj - fixed)**:
```xml
<PropertyGroup>
    <TargetFrameworks>net9.0-android;net9.0-ios;net9.0-maccatalyst;net9.0-browserwasm;net9.0-desktop;net9.0</TargetFrameworks>
</PropertyGroup>
```

**Common Mistakes**:
- Alphabetically sorting TFMs during code review or auto-formatting
- Removing all platform TFMs temporarily for debugging and forgetting to restore the order
- Adding `net9.0` to the front when creating a NuGet package project that also builds the app
- Not restarting Visual Studio after fixing the order (VS2022 caches the TFM analysis)

**Uno Platform Notes**: After fixing TFM order, close and reopen the solution in VS2022. IntelliSense may continue to show stale errors until the project is fully re-evaluated. Running `dotnet build` from the command line first confirms the fix is correct before troubleshooting VS2022.

**Reference**: https://platform.uno/docs/articles/troubleshooting-build-errors.html

---

### Resolve "MonoAOTCompiler Task Not Found" and UNOWA0001

**Rule**: When the build fails with "The MonoAOTCompiler task was not found" or warning UNOWA0001 about missing wasm-tools, install the `wasm-tools` workload for your current .NET SDK version.

**Why**: WebAssembly builds require the `wasm-tools` .NET workload, which provides the Mono AOT compiler and wasm-opt binary. This workload is not installed by default with the .NET SDK. Each .NET major version (8, 9) has its own wasm-tools version, and upgrading the SDK does not carry over previously installed workloads.

**Example (diagnosis and fix)**:
```bash
# Check currently installed workloads
dotnet workload list

# If wasm-tools is missing or shows wrong version:
dotnet workload install wasm-tools

# Verify after install
dotnet workload list
# Should show: wasm-tools    9.0.x/9.0.100    SDK 9.0.xxx

# Alternative: use uno-check for comprehensive validation
dotnet tool update -g uno.check
uno-check
```

**Example (CI/CD fix - Azure DevOps)**:
```yaml
steps:
  - task: UseDotNet@2
    inputs:
      version: '9.0.x'
  - script: dotnet workload install wasm-tools
    displayName: 'Install wasm-tools workload'
  - script: dotnet build -f net9.0-browserwasm
    displayName: 'Build WebAssembly'
```

**Common Mistakes**:
- Installing `wasm-tools` with an older .NET SDK on the PATH (check `dotnet --version` first)
- Not adding the workload install step to CI/CD pipelines (local builds work, CI fails)
- Confusing `wasm-tools` with `wasm-experimental` (different workloads with different contents)
- Running `dotnet workload repair` instead of `dotnet workload install` (repair only fixes corruption, it does not add missing workloads)

**Uno Platform Notes**: The `uno-check` tool automatically detects missing workloads and offers to install them. It is the single best diagnostic tool for environment issues. Run it after every .NET SDK upgrade: `uno-check`.

**Reference**: https://platform.uno/docs/articles/external/uno.check/doc/using-uno-check.html

---

### Fix Android Build Failures: Java Heap, SDK Mismatch, and Binding Errors

**Rule**: For Android build failures, check these three causes in order: (1) Java heap space exhaustion, (2) Android SDK version mismatch, (3) Java binding generation errors.

**Why**: Android builds invoke the Java compiler (javac), Android Asset Packaging Tool (aapt2), and D8/R8 dexing tools, all of which run in JVM processes. These tools fail with unhelpful error messages when they hit memory limits, use wrong SDK versions, or encounter conflicting Java bindings. Checking in the recommended order resolves 90% of Android build issues.

**Example (Java heap space fix)**:
```xml
<!-- Add to csproj inside an Android-specific Choose block -->
<Choose>
    <When Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">
        <PropertyGroup>
            <JavaMaximumHeapSize>1g</JavaMaximumHeapSize>
            <!-- Increase to 2g for very large projects -->
        </PropertyGroup>
    </When>
</Choose>
```

**Example (SDK version check and fix)**:
```bash
# Run uno-check to validate Android SDK setup
uno-check

# Common output when SDK is misconfigured:
# [ERROR] Android SDK Platform 34 is not installed
# [ERROR] Android Build Tools 34.0.0 is not installed
# Fix: Accept the uno-check prompt to install missing components
```

**Example (binding error diagnosis)**:
```bash
# Clean and rebuild to clear stale binding caches
dotnet clean MyApp.csproj -f net9.0-android
dotnet build MyApp.csproj -f net9.0-android

# If errors persist, check for namespace conflicts between
# your Java bindings and Android SDK classes
```

**Common Mistakes**:
- Setting `JavaMaximumHeapSize` globally instead of only for Android TFM (wastes memory on other builds)
- Ignoring `uno-check` output about outdated Android SDK components
- Not running `dotnet clean` before rebuilding after changing Android SDK versions (stale intermediate files)
- Having conflicting versions of `Xamarin.AndroidX.*` packages that produce duplicate class errors

**Uno Platform Notes**: The Uno.Sdk manages most Android SDK dependencies automatically. If you see `Xamarin.AndroidX` version conflicts, check that you are not manually referencing AndroidX packages that the Uno.Sdk already includes. Remove explicit AndroidX references and let the SDK resolve them.

**Reference**: https://platform.uno/docs/articles/troubleshooting-android.html

---

### Fix iOS Build Failures: Provisioning, Linker, and Simulator Architecture

**Rule**: For iOS build failures, check (1) provisioning profile validity, (2) linker configuration for reflection-heavy code, and (3) correct TFM for simulator vs. device builds.

**Why**: iOS builds have strict code signing requirements, an aggressive managed linker that removes "unused" code, and separate architectures for simulator (x64/arm64 host) vs. device (arm64). Each of these produces different error categories that are easy to conflate. Checking in order avoids wasting time on linker issues when the real problem is an expired provisioning profile.

**Example (provisioning profile check)**:
```xml
<!-- Platforms/iOS/Entitlements.plist must be valid -->
<!-- Verify in Apple Developer Portal that: -->
<!-- 1. Your App ID matches the Bundle Identifier in Info.plist -->
<!-- 2. The provisioning profile is not expired -->
<!-- 3. The signing certificate is installed in Keychain -->

<!-- For development builds, use automatic signing: -->
<PropertyGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">
    <CodesignKey>Apple Development</CodesignKey>
    <CodesignProvision>Automatic</CodesignProvision>
</PropertyGroup>
```

**Example (linker configuration for reflection)**:
```csharp
// Types accessed via reflection (e.g., DI, serialization) must be preserved
// Add [Preserve] attribute or use a linker descriptor XML

// Option 1: Attribute on the type
[Preserve(AllMembers = true)]
public class MyService : IMyService
{
    // All members preserved from linker stripping
}

// Option 2: Linker descriptor file (LinkerDescriptor.xml)
```

```xml
<!-- LinkerDescriptor.xml -->
<linker>
    <assembly fullname="MyApp">
        <type fullname="MyApp.Services.MyService" preserve="all" />
    </assembly>
</linker>

<!-- Reference in csproj -->
<ItemGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">
    <LinkDescription Include="LinkerDescriptor.xml" />
</ItemGroup>
```

**Example (simulator TFM)**:
```xml
<!-- For Debug builds targeting the iOS Simulator -->
<PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TargetFrameworks>net9.0-iossimulator;net9.0-android;net9.0-browserwasm;net9.0-desktop;net9.0</TargetFrameworks>
</PropertyGroup>

<!-- For Release builds targeting physical devices -->
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TargetFrameworks>net9.0-ios;net9.0-android;net9.0-browserwasm;net9.0-desktop;net9.0</TargetFrameworks>
</PropertyGroup>
```

**Common Mistakes**:
- Building with `net9.0-ios` TFM and deploying to the simulator (architecture mismatch)
- Setting the linker to `None` globally to avoid stripping errors (produces an oversized app that may be rejected by the App Store)
- Not testing with `SdkOnly` linker mode first (strips only SDK assemblies, preserving all app code)
- Forgetting to renew provisioning profiles before they expire, causing CI/CD failures

**Uno Platform Notes**: Uno Platform uses the standard .NET iOS build pipeline. All iOS-specific troubleshooting from Microsoft's .NET MAUI documentation also applies to Uno Platform iOS builds. The `[Preserve]` attribute is from `Foundation` namespace on iOS.

**Reference**: https://platform.uno/docs/articles/troubleshooting-ios.html

---

### Resolve Visual Studio 2022 Issues: IntelliSense, Hot Reload, and XAML Designer

**Rule**: For VS2022 issues, apply these specific fixes: (1) IntelliSense false errors - check TFM order and restart VS, (2) Hot Reload not working - set `DOTNET_MODIFIABLE_ASSEMBLIES=debug`, (3) blank XAML designer - use Hot Design or Hot Reload instead.

**Why**: VS2022's integration with multi-TFM projects is imperfect. IntelliSense evaluates against the first TFM, Hot Reload requires an environment variable that VS2022 does not always set automatically, and the WinUI XAML designer does not render Uno Platform cross-platform controls. These are known limitations with specific workarounds, not bugs to chase.

**Example (IntelliSense fix - TFM ordering)**:
```xml
<!-- Ensure a platform TFM is first for correct IntelliSense -->
<TargetFrameworks>net9.0-desktop;net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0</TargetFrameworks>
<!-- Using net9.0-desktop first gives the best IntelliSense experience -->
<!-- because Desktop/Skia supports the widest range of Uno Platform APIs -->
```

**Example (Hot Reload environment variable)**:
```powershell
# Set in your terminal before launching the app
$env:DOTNET_MODIFIABLE_ASSEMBLIES = "debug"

# Or add to your launchSettings.json:
```

```json
{
    "profiles": {
        "MyApp (Desktop)": {
            "commandName": "Project",
            "environmentVariables": {
                "DOTNET_MODIFIABLE_ASSEMBLIES": "debug"
            }
        }
    }
}
```

**Example (Hot Design as XAML designer replacement)**:
```
1. Build and run the app (F5)
2. The Hot Design toolbar appears in the running app
3. Click the Hot Design button to enter design mode
4. Edit XAML in VS2022 - changes appear in real time
5. No separate XAML designer window needed
```

**Common Mistakes**:
- Restarting VS2022 repeatedly for IntelliSense issues without checking TFM order first
- Setting `DOTNET_MODIFIABLE_ASSEMBLIES=debug` in system environment variables (only needed per-session for development)
- Expecting the VS2022 XAML designer to render Uno Platform controls (it only works for WinUI/Windows target)
- Disabling the XAML designer extension thinking it causes issues (it is benign and needed for XAML editing features)

**Uno Platform Notes**: For the best development experience, use `net9.0-desktop` as the first TFM. The Desktop/Skia target gives the fastest build times, broadest API coverage for IntelliSense, and supports Hot Reload without additional configuration. Switch to other TFMs only when testing platform-specific behavior.

**Reference**: https://platform.uno/docs/articles/features/working-with-xaml-hot-reload.html

---

### Diagnose Wasm Runtime Issues: Blank Screen, CORS, and MIME Types

**Rule**: When a WebAssembly app shows a blank screen, debug in this order: (1) open browser console (F12) for JavaScript errors, (2) check for CORS errors on API calls, (3) verify the hosting server serves `.wasm` files with `application/wasm` MIME type.

**Why**: WebAssembly apps run inside the browser sandbox, which enforces strict security policies. A blank screen typically means the bootstrapper failed to load, the .wasm binary was blocked by CORS or MIME type restrictions, or a startup exception occurred before the UI rendered. The browser console is the only diagnostic tool available - there are no server logs for client-side issues.

**Example (browser console diagnosis)**:
```
// Common console errors and their meanings:

// 1. "Failed to load resource: net::ERR_FAILED" on .wasm file
//    Fix: Server must serve .wasm with Content-Type: application/wasm

// 2. "Access to fetch at '...' from origin '...' has been blocked by CORS policy"
//    Fix: Configure CORS headers on API server

// 3. "Unhandled exception rendering component" or "Error: Mono runtime init failed"
//    Fix: Check for missing assemblies or startup exceptions
```

**Example (server MIME type configuration - IIS web.config)**:
```xml
<system.webServer>
    <staticContent>
        <remove fileExtension=".wasm" />
        <mimeMap fileExtension=".wasm" mimeType="application/wasm" />
        <remove fileExtension=".clr" />
        <mimeMap fileExtension=".clr" mimeType="application/octet-stream" />
        <remove fileExtension=".pdb" />
        <mimeMap fileExtension=".pdb" mimeType="application/octet-stream" />
        <remove fileExtension=".dat" />
        <mimeMap fileExtension=".dat" mimeType="application/octet-stream" />
        <remove fileExtension=".blat" />
        <mimeMap fileExtension=".blat" mimeType="application/octet-stream" />
    </staticContent>
</system.webServer>
```

**Example (CORS configuration - ASP.NET backend)**:
```csharp
// In Program.cs of your API project
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://myapp.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

app.UseCors();
```

**Common Mistakes**:
- Not checking the browser console at all (the blank screen has no visual error indicator)
- Deploying to a server without configuring MIME types for `.wasm`, `.clr`, `.dat`, and `.blat` files
- Using `*` for CORS origins in production (security risk)
- Assuming the app is broken when the issue is a slow initial download of the .wasm binary (add a loading indicator in `index.html`)
- Not enabling brotli/gzip compression on the server for `.wasm` files (they compress extremely well, reducing load time by 70-80%)

**Uno Platform Notes**: The Uno Platform Wasm bootstrapper uses several custom file extensions (`.clr`, `.dat`, `.blat`) for assembly and resource files. All of these must be served with an appropriate MIME type. The `application/octet-stream` fallback works for non-wasm files. Check the bootstrapper documentation for the complete list of required MIME types.

**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-bootstrapper.html

---

### Use Diagnostic Tools Systematically

**Rule**: When encountering any build or runtime error, run the appropriate diagnostic tool before searching for solutions: `uno-check` for environment issues, `adb logcat` for Android runtime crashes, browser F12 console for Wasm issues, and the VS2022 Output window for build diagnostics.

**Why**: Most Uno Platform errors have well-documented solutions, but the error messages do not always point directly to the fix. Diagnostic tools provide structured output that narrows the problem space faster than reading error messages in isolation. `uno-check` alone resolves roughly 60% of new developer setup issues by detecting missing workloads, wrong SDK versions, and misconfigured Android/iOS SDKs.

**Example (diagnostic tool reference)**:

| Symptom | Tool | Command/Action |
|---|---|---|
| Build fails on any platform | `uno-check` | `dotnet tool update -g uno.check && uno-check` |
| Wasm build fails | `dotnet workload list` | Verify `wasm-tools` is installed for correct SDK |
| Android runtime crash | `adb logcat` | `adb logcat -s mono-rt dotnet *:E` |
| iOS runtime crash | Xcode Console | Open Devices and Simulators, check device logs |
| Wasm blank screen | Browser F12 | Check Console tab for JavaScript/Mono errors |
| VS2022 IntelliSense wrong | VS Output Window | Switch to "Build" output, check for hidden warnings |
| NuGet restore failures | `dotnet restore -v detailed` | Check for source/auth issues |
| Hot Reload not applying | VS Output Window | Look for "Hot Reload" entries indicating file change detection |

**Example (uno-check comprehensive run)**:
```bash
# Update to latest version
dotnet tool update -g uno.check

# Run full environment validation
uno-check

# Expected output categories:
# - .NET SDK version
# - Required workloads (wasm-tools, maui, etc.)
# - Android SDK components
# - iOS/macOS build tools (on macOS)
# - Uno.Sdk version compatibility
# - Visual Studio version (on Windows)
```

**Example (adb logcat filtered for Uno Platform)**:
```bash
# Filter for .NET runtime errors on Android
adb logcat -s mono-rt dotnet *:E

# Filter for unhandled exceptions
adb logcat | grep -i "unhandled"

# Clear log and start fresh
adb logcat -c && adb logcat -s mono-rt dotnet *:E
```

**Common Mistakes**:
- Searching Stack Overflow before running `uno-check` (wastes time on environment issues that have automated fixes)
- Not updating `uno-check` before running it (older versions may not detect newer SDK requirements)
- Reading only the first error in a long build log (often the root cause is buried in the middle)
- Not using the VS2022 Output window's "Build" source (the Error List window sometimes misses warnings that explain the root cause)
- Ignoring yellow warnings in the build output (they often predict the next error you will hit)

**Uno Platform Notes**: The `uno-check` tool is maintained by the Uno Platform team and is updated with each Uno.Sdk release. It is the authoritative tool for validating your development environment. When filing bug reports on GitHub, always include the full `uno-check` output.

**Reference**: https://platform.uno/docs/articles/external/uno.check/doc/using-uno-check.html
