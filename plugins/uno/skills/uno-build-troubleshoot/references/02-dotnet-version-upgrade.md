# .NET Version Upgrade Reference

Detailed patterns for upgrading Uno Platform projects between .NET versions, handling Uno.Extensions breaking changes, and migrating the WebAssembly bootstrapper.

---

### Update Target Frameworks from net8.0 to net9.0

**Rule**: Change all `net8.0-*` TFMs to `net9.0-*` in the csproj `<TargetFrameworks>` element, preserving the correct ordering with platform-specific TFMs before the base TFM.

**Why**: .NET 9 introduces new BCL APIs, improved AOT compilation for WebAssembly, updated Android/iOS SDK bindings, and performance improvements in the Skia renderer. Uno Platform 5.5+ requires .NET 9 for new features and will eventually drop .NET 8 support. Delaying the upgrade increases the gap and makes the eventual migration harder.

**Example (csproj - before)**:
```xml
<PropertyGroup>
    <TargetFrameworks>net8.0-android;net8.0-ios;net8.0-maccatalyst;net8.0-browserwasm;net8.0-desktop;net8.0</TargetFrameworks>
</PropertyGroup>
```

**Example (csproj - after)**:
```xml
<PropertyGroup>
    <TargetFrameworks>net9.0-android;net9.0-ios;net9.0-maccatalyst;net9.0-browserwasm;net9.0-desktop;net9.0</TargetFrameworks>
</PropertyGroup>
```

**Common Mistakes**:
- Changing only some TFMs (e.g., updating `net8.0-android` but forgetting `net8.0-ios`), causing mixed-version build failures
- Reordering TFMs during the update and accidentally placing `net9.0` first (causes UNOB0011/UNOB0013)
- Not updating class library projects that are referenced by the app project, leaving them on `net8.0`
- Forgetting to update `net8.0-iossimulator` to `net9.0-iossimulator` in Debug configurations

**Uno Platform Notes**: The TFM `net9.0-browserwasm` is specific to Uno Platform. Standard .NET Wasm projects use `net9.0-browser`, but Uno Platform requires `net9.0-browserwasm` for the Uno bootstrapper to activate correctly.

**Reference**: https://platform.uno/docs/articles/migrating-from-net8-to-net9.html

---

### Update global.json Uno.Sdk Version

**Rule**: After changing TFMs, update the `Uno.Sdk` version in `global.json` to a release that supports the new .NET version; always check the Uno Platform release notes for the minimum required SDK version.

**Why**: The Uno.Sdk version controls which MSBuild targets, source generators, and platform integrations are used. An older Uno.Sdk may not recognize `net9.0-*` TFMs, causing immediate build failure with "The target framework is not supported" errors. The Uno.Sdk version and the .NET version must be compatible.

**Example (global.json - before)**:
```json
{
    "msbuild-sdks": {
        "Uno.Sdk": "5.2.175"
    }
}
```

**Example (global.json - after)**:
```json
{
    "msbuild-sdks": {
        "Uno.Sdk": "5.6.22"
    }
}
```

**Common Mistakes**:
- Updating TFMs to `net9.0` but leaving the Uno.Sdk on a version that only supports `net8.0`
- Using a preview/nightly Uno.Sdk version in production without realizing it (check for `-dev` or `-preview` suffixes)
- Forgetting that `global.json` also pins the .NET SDK version (if a `sdk` section exists), which can conflict with the new .NET version
- Not running `dotnet restore` after changing `global.json`, causing stale SDK resolution

**Uno Platform Notes**: Check https://github.com/nickvdyck/uno-check or run `uno-check` to verify that the installed Uno.Sdk version matches your global.json. The `uno-check` tool also validates that the .NET SDK, workloads, and Android/iOS SDKs are compatible.

**Reference**: https://platform.uno/docs/articles/external/uno.sdk/doc/articles/overview.html

---

### Install or Update the wasm-tools Workload

**Rule**: After every .NET major version upgrade, run `dotnet workload install wasm-tools` to install the WebAssembly build tools matching the new SDK version.

**Why**: The `wasm-tools` workload provides the MonoAOTCompiler, wasm-opt, and other build tools required for WebAssembly compilation. Each .NET major version ships a new version of these tools. Without the matching workload, builds fail with "The MonoAOTCompiler task was not found" (build error) or "UNOWA0001: Native WebAssembly assets were detected, but the wasm-tools workload could not be located" (SDK warning that becomes an error).

**Example (CLI)**:
```bash
# Install for the current .NET SDK
dotnet workload install wasm-tools

# Verify installation
dotnet workload list

# Expected output should include:
# wasm-tools   9.0.x   SDK 9.0.xxx
```

**Example (CI/CD - GitHub Actions)**:
```yaml
steps:
  - uses: actions/setup-dotnet@v4
    with:
      dotnet-version: '9.0.x'
  - run: dotnet workload install wasm-tools
  - run: dotnet build -f net9.0-browserwasm
```

**Common Mistakes**:
- Assuming `wasm-tools` persists across .NET SDK updates (it does not; each major version needs its own install)
- Running `dotnet workload install wasm-tools` with an older SDK on the PATH, installing tools for the wrong .NET version
- Not adding the workload install step to CI/CD pipelines, causing builds to succeed locally but fail in CI
- Confusing `wasm-tools` with `wasm-experimental` (different workload with additional preview features)

**Uno Platform Notes**: Run `uno-check` after upgrading. It validates that `wasm-tools` is installed for the correct SDK version and reports mismatches before you encounter cryptic build errors.

**Reference**: https://platform.uno/docs/articles/external/uno.check/doc/using-uno-check.html

---

### Handle Uno.Extensions 5.2+ Breaking Changes for MSAL

**Rule**: When upgrading to Uno.Extensions 5.2 or later, update all `AddMsal()` calls to pass the `Window` parameter; the parameterless overload has been removed.

**Why**: MSAL (Microsoft Authentication Library) requires a window handle for interactive authentication flows (token acquisition, consent prompts). In earlier versions, Uno.Extensions resolved the window implicitly, which was unreliable on some platforms. The 5.2 release made the Window parameter mandatory to ensure consistent behavior across Android, iOS, Desktop, and WebAssembly.

**Example (C# - before, Uno.Extensions < 5.2)**:
```csharp
host.UseMsal(msal => msal
    .WithClientId("your-client-id")
    .WithRedirectUri("your-redirect-uri"));
```

**Example (C# - after, Uno.Extensions 5.2+)**:
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var builder = this.CreateBuilder(args)
        .Configure(host => host
            .UseAuthentication(auth =>
                auth.AddMsal(msal => msal
                    .WithClientId("your-client-id")
                    .WithRedirectUri("your-redirect-uri"),
                    // Window parameter now required
                    configureAcquireTokenInteractive:
                        (provider, builder) => builder.WithParentActivityOrWindow(provider.GetRequiredService<Window>())
                )
            )
        );

    MainWindow = builder.Window;
}
```

**Common Mistakes**:
- Not reading the Uno.Extensions 5.2 release notes before upgrading, leading to surprise compile errors
- Passing `null` for the Window parameter to suppress the compile error, which causes NullReferenceException at runtime during authentication
- Forgetting to register the `Window` in the DI container (the Uno.Sdk builder does this automatically, but custom DI setups may not)
- Upgrading Uno.Extensions without upgrading the Uno.Sdk, causing version mismatch errors

**Uno Platform Notes**: The Window parameter requirement applies to all platforms. On WebAssembly, the Window object represents the browser tab context. On Android, it resolves to the current Activity. Ensure your DI container has the Window registered before MSAL initialization runs.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Authentication/HowToMsalAuthentication.html

---

### Adapt to Navigation and Theme Breaking Changes in 5.2

**Rule**: Review and update navigation route registrations and theme resource references when upgrading to Uno.Extensions 5.2+; some APIs have changed signatures and Uno Themes 5.1+ uses different color naming conventions.

**Why**: The navigation framework in 5.2 refined route registration to support nested navigation and deep linking more reliably. Some method signatures changed to accept additional parameters. Separately, Uno Themes 5.1 aligned color resource names with Material Design 3 token naming, so hardcoded resource references like `{StaticResource MaterialPrimaryBrush}` may resolve to `null` if the name changed.

**Example (Navigation - before)**:
```csharp
// Older route registration
routes.Register(
    new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(),
        Nested: new RouteMap[]
        {
            new RouteMap("Details", View: views.FindByViewModel<DetailsViewModel>())
        }
    )
);
```

**Example (Navigation - after, verify against generated reference project)**:
```csharp
// Updated route registration - always compare with dotnet new unoapp output
routes.Register(
    new RouteMap("Main", View: views.FindByViewModel<MainViewModel>(),
        Nested: new RouteMap[]
        {
            new RouteMap("Details", View: views.FindByViewModel<DetailsViewModel>())
        }
    )
);
// Note: The structure may look similar, but check for new overloads,
// required parameters, and renamed methods in the release notes.
```

**Example (Themes - color resource name changes)**:
```xml
<!-- Before (Uno Themes < 5.1) -->
<SolidColorBrush x:Key="CustomBrush" Color="{StaticResource MaterialPrimaryColor}" />

<!-- After (Uno Themes 5.1+) - verify exact name in ColorPaletteOverride.xaml -->
<SolidColorBrush x:Key="CustomBrush" Color="{ThemeResource PrimaryColor}" />
```

**Common Mistakes**:
- Upgrading Uno.Extensions without checking the release notes for breaking changes
- Assuming theme resource names are stable across major versions
- Not testing navigation flows end-to-end after upgrading (deep links, back navigation, dialog dismissal)
- Hardcoding Material color resource names instead of using `ThemeResource` references that adapt to theme changes

**Uno Platform Notes**: Always generate a fresh `dotnet new unoapp` after a major Uno.Extensions upgrade and diff the `App.xaml`, `App.xaml.cs`, and theme resource dictionaries against your existing code. This is the fastest way to find all breaking changes.

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Overview/Navigation/NavigationOverview.html

---

### Migrate the Wasm Bootstrapper from 8.x to 9.x

**Rule**: When upgrading from Uno Wasm Bootstrapper 8.x to 9.x, switch the SDK, remove the DevServer package, and replace all deprecated MSBuild properties with their 9.x equivalents.

**Why**: The Uno Wasm Bootstrapper 9.x integrated the dev server into the main SDK, eliminating the separate `Uno.Wasm.Bootstrap.DevServer` package. Several MSBuild properties were renamed or removed to align with the new .NET 9 WebAssembly hosting model. Leaving deprecated properties causes build warnings that will become errors in future releases.

**Example (csproj - before, bootstrapper 8.x)**:
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    <PropertyGroup>
        <WasmShellMode>browser</WasmShellMode>
        <WasmShellGenerateCompressedFiles>true</WasmShellGenerateCompressedFiles>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Uno.Wasm.Bootstrap" Version="8.x.x" />
        <PackageReference Include="Uno.Wasm.Bootstrap.DevServer" Version="8.x.x" />
    </ItemGroup>
</Project>
```

**Example (csproj - after, Uno.Sdk Single Project)**:
```xml
<Project Sdk="Uno.Sdk">
    <PropertyGroup>
        <TargetFrameworks>net9.0-browserwasm;net9.0-desktop;net9.0</TargetFrameworks>
    </PropertyGroup>
    <!-- DevServer package is no longer needed -->
    <!-- WasmShellMode and WasmShellGenerateCompressedFiles are handled by the SDK -->
</Project>
```

**Common Mistakes**:
- Keeping `Uno.Wasm.Bootstrap.DevServer` in the package references (causes version conflict warnings)
- Retaining deprecated `WasmShell*` properties that the 9.x SDK no longer reads, producing silent misconfigurations
- Not removing the explicit `Microsoft.NET.Sdk.Web` SDK reference when migrating to `Uno.Sdk` (the Uno.Sdk handles web hosting internally)
- Forgetting to update `wwwroot/index.html` if it references old bootstrapper script paths

**Uno Platform Notes**: The Uno.Sdk abstracts the Wasm bootstrapper entirely. If you are using Single Project with Uno.Sdk, you do not need to reference bootstrapper packages at all. The SDK handles bootstrapper version management internally based on the `Uno.Sdk` version in `global.json`.

**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-bootstrapper.html

---

### Remove Deprecated Properties and Packages Systematically

**Rule**: After upgrading, search the entire solution for deprecated MSBuild properties, removed NuGet packages, and obsolete API calls; do not rely solely on build errors to find them.

**Why**: Many deprecated properties produce warnings rather than errors, so builds appear to succeed while silently using wrong defaults. Deprecated APIs marked with `[Obsolete]` compile with warnings that are easy to miss in verbose build output. A systematic search catches these before they cause runtime issues or become hard errors in the next release.

**Example (search checklist)**:
```bash
# Search for deprecated Wasm properties
# In your csproj files, look for:
#   WasmShellMode
#   WasmShellGenerateCompressedFiles
#   WasmShellEnableEmscriptenWindows
#   MonoRuntimeDebuggerEnabled

# Search for removed packages
# In your csproj files, look for:
#   Uno.Wasm.Bootstrap.DevServer
#   Uno.UI.RemoteControl (replaced by built-in Hot Reload)
#   Uno.UI.Adapter.Microsoft.Extensions.Logging (now built into Uno.Sdk)

# Search for obsolete API usage
# In your C# files, look for:
#   Windows.UI.Xaml (should be Microsoft.UI.Xaml for WinUI 3)
#   Dispatcher.RunAsync (should be DispatcherQueue.TryEnqueue)
```

**Example (csproj - cleaning deprecated properties)**:
```xml
<!-- REMOVE these deprecated properties -->
<!--
<WasmShellMode>browser</WasmShellMode>
<MonoRuntimeDebuggerEnabled>true</MonoRuntimeDebuggerEnabled>
<WasmShellEnableEmscriptenWindows>true</WasmShellEnableEmscriptenWindows>
-->

<!-- KEEP these valid properties -->
<WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
```

**Common Mistakes**:
- Only checking the main app csproj and missing deprecated properties in class library projects
- Ignoring build warnings because the build succeeds (warnings often indicate upcoming breaking changes)
- Not running `dotnet restore --force` after removing packages, leaving stale entries in the NuGet cache
- Skipping `uno-check` after the upgrade, missing environment-level issues (wrong SDK version, missing workloads)

**Uno Platform Notes**: The `uno-check` tool validates your environment against the requirements of the installed Uno.Sdk version. Always run it after a major version upgrade: `dotnet tool update -g uno.check && uno-check`.

**Reference**: https://platform.uno/docs/articles/external/uno.check/doc/using-uno-check.html
