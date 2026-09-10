# Setup and Bootstrapper Reference

Detailed reference for WebAssembly project setup and Uno.Wasm.Bootstrap configuration in Uno Platform 9.x+.

---

### Install the wasm-tools Workload
**Rule**: Always install the `wasm-tools` workload before building a WebAssembly project.
**Why**: The .NET WebAssembly runtime, AOT compiler, and native linking toolchain are all delivered through this workload. Without it, builds fail with error UNOWA0001: "Native WebAssembly assets were detected, but the wasm-tools workload could not be located."
**Example (CLI)**:
```bash
# For .NET 9 as the primary SDK
dotnet workload install wasm-tools

# If .NET SDK 10.0 is installed but you target net9.0
dotnet workload install wasm-tools-net9

# Verify installation
dotnet workload list
```
**Common Mistakes**:
- Forgetting to restart Visual Studio after installing workloads (VS caches workload state at launch).
- Running `dotnet workload install` from a global directory instead of the project folder, which can select the wrong SDK band.
- Ignoring the `uno-check` tool, which validates all prerequisites including workloads: `dotnet tool install -g uno.check && uno-check`.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/using-the-bootstrapper.html

---

### Use Microsoft.NET.Sdk.WebAssembly as the Project SDK
**Rule**: Set the project SDK to `Microsoft.NET.Sdk.WebAssembly` for bootstrapper 9.x projects.
**Why**: Bootstrapper 9.x moved from `Microsoft.NET.Sdk.Web` to `Microsoft.NET.Sdk.WebAssembly`. Using the old SDK causes build failures because the WebAssembly-specific MSBuild targets and props are not imported. The `Uno.Wasm.Bootstrap.DevServer` package is also no longer needed - the dev server is built into the new SDK.
**Example (csproj)**:
```xml
<Project Sdk="Microsoft.NET.Sdk.WebAssembly">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0-browserwasm</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Uno.Wasm.Bootstrap" Version="9.0.*" />
    <!-- DevServer package is NO LONGER needed in 9.x -->
  </ItemGroup>

</Project>
```
If using the Uno.Sdk (recommended), update to Uno.Sdk 5.5 or later:
```xml
<Project Sdk="Uno.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net9.0-browserwasm;net9.0-desktop;...</TargetFrameworks>
  </PropertyGroup>
</Project>
```
**Common Mistakes**:
- Keeping `Sdk="Microsoft.NET.Sdk.Web"` after upgrading to bootstrapper 9.x.
- Leaving the `Uno.Wasm.Bootstrap.DevServer` package in the project file (it is unnecessary and can cause conflicts).
- Using `net9.0` instead of `net9.0-browserwasm` as the TFM.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/using-the-bootstrapper.html

---

### Bootstrapper Version Mapping
**Rule**: Match the bootstrapper major version to the target .NET runtime version.
**Why**: Each major version of `Uno.Wasm.Bootstrap` targets a specific .NET runtime. Mismatches cause runtime initialization failures because the bootstrapper loads the wrong runtime binaries.
**Example (version map)**:
| Bootstrapper | .NET Runtime | Notes |
|---|---|---|
| 8.x | .NET 8 | Last version with `Microsoft.NET.Sdk.Web` |
| 9.x | .NET 9 | Uses `Microsoft.NET.Sdk.WebAssembly`; IDBFS off by default |
| 10.x | .NET 10 | Targets .NET 10 runtime |
| 7.x | .NET 7 | Legacy |
| 3.x | .NET 6 | Legacy |
| 2.x | Mono | Legacy |

```xml
<!-- Correct: bootstrapper 9.x with .NET 9 TFM -->
<TargetFramework>net9.0-browserwasm</TargetFramework>
<PackageReference Include="Uno.Wasm.Bootstrap" Version="9.0.*" />

<!-- WRONG: bootstrapper 9.x with .NET 8 TFM -->
<TargetFramework>net8.0-browserwasm</TargetFramework>
<PackageReference Include="Uno.Wasm.Bootstrap" Version="9.0.*" />
```
**Common Mistakes**:
- Using bootstrapper 9.x with a `net8.0-browserwasm` TFM.
- Upgrading the TFM to .NET 9 but leaving the bootstrapper at version 8.x.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/using-the-bootstrapper.html

---

### Restore IDBFS Support in 9.x
**Rule**: Explicitly opt in to IDBFS when migrating from bootstrapper 8.x to 9.x.
**Why**: In bootstrapper 8.x and earlier, the Emscripten IDBFS (IndexedDB-backed filesystem) was enabled by default. In 9.x, the .NET 9 interpreter runtime does not enable it. Applications that persist files to the virtual filesystem (e.g., SQLite databases, cached data) will silently fail to persist without this configuration.
**Example (csproj)**:
```xml
<ItemGroup>
    <WasmShellExtraEmccFlags Include="-lidbfs.js" />
    <WasmShellEmccExportedRuntimeMethod Include="IDBFS" />
</ItemGroup>
```
**Common Mistakes**:
- Assuming IDBFS "just works" after upgrading from 8.x, then discovering data loss at runtime.
- Adding only one of the two required lines (both the `-lidbfs.js` flag and the exported runtime method are required).
- Confusing IDBFS (Emscripten's IndexedDB filesystem) with direct IndexedDB JavaScript interop.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-idbfs.html

---

### Migrate Deprecated 8.x Properties
**Rule**: Remove all deprecated 8.x MSBuild properties when upgrading to bootstrapper 9.x.
**Why**: The .NET SDK now handles functionality that was previously managed by the bootstrapper. Keeping deprecated properties causes build warnings and can mask real configuration issues. The .NET SDK handles compression, obfuscation (via WebCIL), Emscripten management, and debugging natively.
**Example (properties to remove)**:
```xml
<!-- REMOVE all of these from your csproj when upgrading to 9.x -->
<!--
<WasmShellOutputPackagePath>...</WasmShellOutputPackagePath>
<WasmShellOutputDistPath>...</WasmShellOutputDistPath>
<WasmShellBrotliCompressionQuality>...</WasmShellBrotliCompressionQuality>
<WasmShellCompressedExtension>...</WasmShellCompressedExtension>
<WasmShellGenerateCompressedFiles>...</WasmShellGenerateCompressedFiles>
<WasmShellEnableEmscriptenWindows>...</WasmShellEnableEmscriptenWindows>
<WasmShellObfuscateAssemblies>...</WasmShellObfuscateAssemblies>
<WasmShellAssembliesExtension>...</WasmShellAssembliesExtension>
<AssembliesFileNameObfuscationMode>...</AssembliesFileNameObfuscationMode>
<MonoRuntimeDebuggerEnabled>...</MonoRuntimeDebuggerEnabled>
<WasmShellEnableAotGSharedVT>...</WasmShellEnableAotGSharedVT>
<WasmShellEnableLongPathSupport>...</WasmShellEnableLongPathSupport>
<WasmShellEnableNetCoreICU>...</WasmShellEnableNetCoreICU>
<WasmShellForceDisableWSL>...</WasmShellForceDisableWSL>
<WasmShellForceUseWSL>...</WasmShellForceUseWSL>
<WashShellGeneratePrefetchHeaders>...</WashShellGeneratePrefetchHeaders>
<WasmShellNinjaAdditionalParameters>...</WasmShellNinjaAdditionalParameters>
<WasmShellPrintAOTSkippedMethods>...</WasmShellPrintAOTSkippedMethods>
<WasmShellPThreadsPoolSize>...</WasmShellPThreadsPoolSize>
<WashShellUseFileIntegrity>...</WashShellUseFileIntegrity>
-->

<!-- REPLACEMENTS for common needs: -->
<!-- Output path: use $(PublishDir) instead of WasmShellOutputPackagePath -->
<!-- Compression: handled by .NET SDK automatically -->
<!-- Obfuscation: .NET SDK uses WebCIL by default -->
<!-- Debugger: .NET SDK manages it; Uno.SDK sets it automatically -->
```
**Common Mistakes**:
- Leaving deprecated properties that silently do nothing, causing confusion when behavior does not match expectations.
- Searching for `WasmShellOutputPackagePath` replacement - use `$(PublishDir)` or the standard `bin/Release/net9.0-browserwasm/publish/wwwroot` path.
- Keeping `Uno.Wasm.Bootstrap.DevServer` as a package reference.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/using-the-bootstrapper.html

---

### Update JS Interop from mono_bind_static_method to getAssemblyExports
**Rule**: Replace `Module.mono_bind_static_method` with `Module.getAssemblyExports` for JavaScript-to-.NET interop.
**Why**: The `mono_bind_static_method` API was removed in .NET 9's Emscripten/runtime update. Existing JS interop code that calls .NET methods from JavaScript will throw runtime errors if not updated.
**Example (JavaScript)**:
```javascript
// OLD (8.x) - no longer works in 9.x
// var myMethod = Module.mono_bind_static_method("[MyAssembly] MyNamespace.MyClass:MyMethod");
// myMethod("hello");

// NEW (9.x) - use getAssemblyExports
const exports = await Module.getAssemblyExports("MyAssembly");
exports.MyNamespace.MyClass.MyMethod("hello");
```
The C# side must use `[JSExport]` attribute on the target method:
```csharp
using System.Runtime.InteropServices.JavaScript;

public partial class MyClass
{
    [JSExport]
    public static string MyMethod(string input)
    {
        return $"Received: {input}";
    }
}
```
**Common Mistakes**:
- Keeping old `mono_bind_static_method` calls that fail silently or throw at runtime.
- Forgetting the `[JSExport]` attribute on the C# method.
- Not making the class `partial` (required for the source generator).
- Using `.bc` native files for Emscripten interop - .NET 9 upgraded to Emscripten 3.1.56 which requires `.a` or `.o` extensions.
**Uno Platform Notes**: Uno Platform's own interop (`Uno.Foundation.WebAssemblyRuntime`) abstracts much of this. Prefer using Uno Platform's built-in interop when available, and reserve raw JS interop for custom scenarios.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/using-the-bootstrapper.html

---

### Threading Is Not Supported on .NET 9 WebAssembly
**Rule**: Do not rely on multi-threading in Uno Platform WebAssembly apps targeting .NET 9.
**Why**: Microsoft's .NET 9 runtime changes prevent managed code from running on the main JavaScript thread when threading is enabled. Since Uno Platform depends on the main thread for UI operations, WebAssembly threading is not supported. Using `Task.Run`, `Thread`, or `ThreadPool` in ways that assume true parallelism will not work as expected - they run on a single thread.
**Example (C#)**:
```csharp
// This works fine - async/await is cooperative, not threaded
await Task.Delay(1000);
var data = await httpClient.GetStringAsync("https://api.example.com/data");

// AVOID - this does NOT create a real background thread on Wasm
// It will execute on the same thread sequentially
await Task.Run(() => HeavyComputation());

// Instead, break heavy work into smaller chunks or use web workers via JS interop
```
**Common Mistakes**:
- Using `ConfigureAwait(false)` expecting thread pool scheduling (there is no thread pool on Wasm).
- Using `Task.Run` to "offload" CPU work from the UI thread (it still blocks the same thread).
- Using `SemaphoreSlim`, `ManualResetEvent`, or other synchronization primitives that assume multiple threads.
**Uno Platform Notes**: The Uno Platform team tracks Microsoft's threading progress. Threading support may return in future .NET 10 preview builds as the runtime team continues development. Design your app's async patterns to be single-threaded safe.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-threading.html
