---
name: uno-build-troubleshoot
description: "Build, upgrade, and runtime troubleshooting for Uno Platform: Single Project migration, .NET version upgrades, Uno.Extensions breaking changes, and diagnosing build and runtime failures. Use when: (1) Migrating to Uno.Sdk Single Project structure, (2) Upgrading .NET or Uno.Sdk versions, (3) Fixing build errors (UNOB0011, UNOB0013, UNOWA0001, MonoAOTCompiler), (4) Diagnosing Android/iOS build failures, (5) Resolving blank Wasm screens or Android startup crashes, (6) Troubleshooting Visual Studio 2022 IntelliSense or Hot Reload problems. Do NOT use for: migrating from WPF (see wpf-to-uno-migration) or Silverlight (see silverlight-to-uno-migration), new project setup (see uno-platform-agent), or Wasm-specific optimization and deployment (see uno-wasm-pwa)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation)"
metadata:
  version: "2.0.0"
---

# Uno Platform Build and Runtime Troubleshooting

Modernizing project structure, moving across .NET versions, and resolving the build
and runtime failures that follow.

## Migration to Single Project (Uno.Sdk 5.2+)

### Overview

Single Project consolidates multiple platform-specific `.csproj` files into one project using `Uno.Sdk`. Migration is optional; existing multi-project structures remain supported.

### Step-by-Step

1. **Create a reference project** with the same name using `dotnet new unoapp`:
   ```bash
   dotnet new unoapp -o MyApp --preset=recommended
   ```

2. **Add `global.json`** next to the solution:
   ```json
   {
       "msbuild-sdks": {
           "Uno.Sdk": "{current Uno.Sdk version}"
       }
   }
   ```

3. **Copy the new `.csproj`** structure from the reference project

4. **Merge platform folders** into `Platforms/`:
   - `MyApp.Wasm/` content to `Platforms/WebAssembly/`
   - `MyApp.Mobile/Android/`, `MyApp.Mobile/iOS/` to `Platforms/`
   - `MyApp.Skia.*` content to `Platforms/Desktop/`

5. **Copy `App.xaml` and `App.xaml.cs`** from reference, then merge your custom changes (OnLaunched, RegisterRoutes, InitializeLogging)

6. **Use conditional TFM blocks** for platform-specific config:
   ```xml
   <Choose>
       <When Condition="'$(TargetFramework)'=='net9.0-browserwasm'">
           <PropertyGroup>
               <WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
           </PropertyGroup>
       </When>
   </Choose>
   ```

7. **Replace `AppHead` with `App`** in all `.cs` files

8. **Delete old platform projects** and folders

### Critical TFM Ordering

```xml
<TargetFrameworks>
    net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop;net9.0
</TargetFrameworks>
```

**Never put `net9.0` first.** VS2022 uses the first TFM for IntelliSense, and `net9.0` alone lacks platform APIs. Putting it first causes UNOB0011 and UNOB0013 errors.

## .NET Version Upgrades

### .NET 8 to .NET 9

1. Update `global.json` to latest Uno.Sdk version
2. Change all TFMs from `net8.0-*` to `net9.0-*`
3. Update `wasm-tools` workload: `dotnet workload install wasm-tools`
4. If using Wasm bootstrapper 8.x, upgrade to 9.x:
   - Change SDK from `Microsoft.NET.Sdk.Web` to `Microsoft.NET.Sdk.WebAssembly`
   - Remove `Uno.Wasm.Bootstrap.DevServer` package
   - Replace deprecated MSBuild properties (see Wasm skill)

### Uno.Extensions 5.2+ Breaking Changes

- **MSAL**: `AddMsal()` now requires a `Window` parameter
- **Navigation**: Some route registration APIs changed
- **Themes**: Uno Themes 5.1+ has color naming changes

## Common Build Errors

### UNOB0011 / UNOB0013

**Cause**: `net9.0` (base TFM) listed first in TargetFrameworks
**Fix**: Move platform TFMs before base TFM:
```xml
net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop;net9.0
```

### "The 'MonoAOTCompiler' task was not found"

**Cause**: Missing `wasm-tools` workload
**Fix**: `dotnet workload install wasm-tools`

### "Native WebAssembly assets were detected, but the wasm-tools workload could not be located" (UNOWA0001)

**Fix**: Run `uno-check` or `dotnet workload install wasm-tools` in the project folder

### Android build failures

- **Java heap space**: Add to `.csproj`:
  ```xml
  <JavaMaximumHeapSize>1g</JavaMaximumHeapSize>
  ```
- **SDK version mismatch**: Run `uno-check` to verify Android SDK setup
- **Binding generation errors**: Clean and rebuild; check for namespace conflicts

### iOS build failures

- **Provisioning profile**: Ensure valid profile in `Platforms/iOS/Entitlements.plist`
- **Linker errors**: Add `[Preserve]` attribute to types consumed by reflection
- **Simulator arch mismatch**: Use `net9.0-iossimulator` TFM for simulator builds

### Visual Studio 2022 Issues

- **IntelliSense errors on valid code**: Restart VS2022; check TFM ordering
- **Hot Reload not working**: Ensure `DOTNET_MODIFIABLE_ASSEMBLIES=debug` is set
- **XAML designer blank**: Expected; use Hot Design or Hot Reload instead
- **Build succeeds but app crashes**: Check Output window for platform-specific errors

## Runtime Issues

### Blank screen on Wasm

1. Check browser console (F12) for JavaScript errors
2. Verify MIME types on hosting server (`application/wasm` required)
3. Ensure `index.html` references correct bootstrapper path

### App crash on Android startup

1. Check `adb logcat` for unhandled exceptions
2. Verify minimum SDK version matches device
3. Check for missing permissions in `AndroidManifest.xml`

### Layout differences across platforms

- Use `SafeArea` for notch/status bar handling
- Test responsive breakpoints on each target size
- Skia renderer may render text metrics slightly differently than native

## Diagnostic Tools

| Tool | Purpose |
|------|---------|
| `uno-check` | Verify environment setup and dependencies |
| `dotnet workload list` | Check installed workloads |
| `adb logcat` | Android runtime logs |
| Browser F12 Console | Wasm runtime errors |
| VS2022 Output Window | Build and runtime diagnostics |

## Common Mistakes

- Migrating by hand instead of using `dotnet new unoapp` as a reference project
- Not updating `global.json` when upgrading Uno.Sdk version
- Keeping deprecated MSBuild properties from older bootstrapper versions
- Ignoring `uno-check` warnings about outdated workloads
- Copying `App.xaml.cs` without merging custom `OnLaunched` logic

## Related Skills

| Skill | Use instead when... |
|-------|-------------------|
| `wpf-to-uno-migration` | Migrating a WPF app to Uno Platform |
| `silverlight-to-uno-migration` | Migrating a Silverlight app to Uno Platform |
| `uno-platform-agent` | Creating new projects (not upgrading existing ones) |
| `uno-extensions-services` | Configuring auth, HTTP, or DI (this skill covers breaking changes in those areas) |
| `uno-wasm-pwa` | Wasm bootstrapper setup, debugging, or deployment (not just upgrade issues) |
| `winui-xaml` | XAML best practices and performance (not build or upgrade issues) |

## Detailed References

- [references/01-single-project-migration.md](references/01-single-project-migration.md) - Read when migrating from multi-project to Single Project structure
- [references/02-dotnet-version-upgrade.md](references/02-dotnet-version-upgrade.md) - Read when upgrading .NET 8 to 9 or updating Uno.Extensions
- [references/03-common-errors.md](references/03-common-errors.md) - Read when encountering build errors, runtime crashes, or platform-specific issues
