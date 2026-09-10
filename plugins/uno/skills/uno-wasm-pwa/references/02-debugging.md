# Debugging Reference

Detailed reference for debugging Uno Platform WebAssembly applications, including Visual Studio setup, browser DevTools, and troubleshooting common failures.

---

### Configure inspectUri in launchSettings.json
**Rule**: Add the `inspectUri` property to every profile in `Properties/launchSettings.json` for C# debugging to work.
**Why**: Visual Studio 2022 uses `inspectUri` to locate the WebAssembly debug proxy. Without it, breakpoints will not bind, and step debugging is impossible. The debug proxy bridges Chrome DevTools Protocol with the .NET runtime's debugging interface.
**Example (launchSettings.json)**:
```json
{
  "profiles": {
    "MyApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "MyApp (IIS Express)": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```
**Common Mistakes**:
- Adding `inspectUri` to only one profile while Visual Studio launches a different one.
- Typos in the template string (e.g., missing `{browserInspectUri}` token).
- Having `"launchBrowser": false` - the browser must launch for the debug proxy to attach.
- Opening browser DevTools (F12) in the app tab while using VS debugging, which interferes with the debug proxy connection.
**Reference**: https://platform.uno/docs/articles/debugging-wasm.html

---

### Use Browser DevTools for Console Output
**Rule**: Use the browser's JavaScript console to view `Console.WriteLine` output from .NET code.
**Why**: In WebAssembly, `Console.WriteLine` is redirected to the browser's JavaScript console via the bootstrapper. This is the primary way to see log output, diagnostic messages, and unhandled exception details during development.
**Example (C#)**:
```csharp
// This output appears in the browser's JavaScript console (F12 > Console tab)
Console.WriteLine("App started successfully");
Console.WriteLine($"Loaded {items.Count} items");

// Exceptions also appear in the console
try
{
    await LoadDataAsync();
}
catch (Exception ex)
{
    Console.WriteLine($"ERROR: {ex.Message}");
    Console.WriteLine(ex.StackTrace);
}
```
**How to view**:
1. Open the browser (Chrome or Edge).
2. Press `F12` to open DevTools.
3. Select the **Console** tab.
4. .NET `Console.WriteLine` output appears as standard log entries.

**Common Mistakes**:
- Searching for output in the Visual Studio Output window (it only appears in the browser console for Wasm).
- Filtering the console to "Errors only" and missing informational output.
- Not checking the console for startup exceptions when the app shows a blank screen.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/using-the-bootstrapper.html

---

### Debug Without VS Using Alt+Shift+D
**Rule**: Use `Alt+Shift+D` in Chrome/Edge for standalone browser-based C# debugging when not using Visual Studio's F5 debugger.
**Why**: This keyboard shortcut activates the .NET WebAssembly debugger directly in the browser, opening a new tab with full source-level debugging. This is useful on macOS/Linux where Visual Studio is not available, or when VS integrated debugging has issues.
**Example (workflow)**:
1. Set the `net9.0-browserwasm` target framework as active.
2. Start without debugging: `Ctrl+F5` in VS, or `dotnet run` from CLI.
3. In the browser tab showing your app, press `Alt+Shift+D`.
4. A new tab opens with the debugger. If Chrome was not launched with the correct flags, the tab shows instructions to relaunch Chrome.
5. In the Sources tab, find your `.cs` files under the assembly name.
6. Set breakpoints and refresh the original app tab if needed.

**Common Mistakes**:
- Pressing `Alt+Shift+D` while also debugging with VS F5 (the two debuggers conflict).
- Not launching Chrome with remote debugging enabled (follow the instructions shown in the debugger tab).
- Having Chrome as primary browser and not wanting to restart it - install Chrome Beta or Chrome Canary as a side-by-side debugging browser.
- Expecting full feature parity with desktop debugging (some features like Edit and Continue are not available).
**Uno Platform Notes**: When debugging in Chrome, `Ctrl+O` in the Sources tab opens a file search dialog, making it easier to locate `.cs` files instead of browsing the folder hierarchy.
**Reference**: https://platform.uno/docs/articles/debugging-wasm.html

---

### Diagnose Blank Screen on Startup
**Rule**: When the app shows a blank screen, check the browser console for .NET runtime errors before investigating XAML issues.
**Why**: A blank screen in a WebAssembly app is almost always caused by a .NET runtime initialization failure, a missing assembly, or a JavaScript error - not a XAML rendering problem. The browser console contains the error that pinpoints the root cause.
**Example (diagnostic checklist)**:
```
1. Open browser DevTools (F12) > Console tab
2. Look for these common errors:

   a) "DllImport unable to load library 'libSkiaSharp'"
      Fix: Run uno-check, verify wasm-tools workload, restart VS.
      If published: serve from bin/Release/net9.0-browserwasm/publish/wwwroot

   b) "Failed to fetch" or "TypeError: Failed to fetch"
      Fix: CORS issue - check the server hosting configuration.
      Or: missing files - verify the publish output is complete.

   c) "Could not load assembly 'MyApp'"
      Fix: Trimming removed a required assembly. Add a LinkerConfig.xml entry.

   d) No errors but blank screen
      Fix: Check if index.html loads. Verify the app's Main() runs by adding
      Console.WriteLine("Main entered") as the first line.

3. Check the Network tab for 404 responses on .wasm or .dll files
4. Verify MIME types are configured on the server (application/wasm for .wasm)
```
**Common Mistakes**:
- Assuming the blank screen is a UI/XAML bug and spending time on layout debugging.
- Not checking the Network tab for failed resource loads.
- Serving published files from the wrong directory (must use `publish/wwwroot`, not `publish` directly).
**Reference**: https://platform.uno/docs/articles/common-issues-wasm.html

---

### Handle CORS for API Calls
**Rule**: Configure CORS headers on your API server when calling endpoints from a WebAssembly app.
**Why**: WebAssembly apps run in the browser and are subject to the Same-Origin Policy. The browser's `fetch` API blocks cross-origin requests unless the server responds with appropriate `Access-Control-Allow-Origin` headers. This is a browser-level restriction, not an Uno Platform limitation.
**Example (common error)**:
```
Access to fetch at 'https://api.example.com/data' from origin 'https://myapp.example.com'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present.
```
**Example (ASP.NET Core server fix)**:
```csharp
// In Program.cs or Startup.cs of your API server
builder.Services.AddCors(options =>
{
    options.AddPolicy("WasmPolicy", policy =>
    {
        policy.WithOrigins("https://myapp.example.com")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

app.UseCors("WasmPolicy");
```
**Common Mistakes**:
- Assuming the CORS error is a bug in the Uno Platform app (it is a server-side configuration issue).
- Using `AllowAnyOrigin()` in production (security risk).
- Forgetting that preflight `OPTIONS` requests also need CORS headers.
- Not testing with the deployed origin URL (localhost and production origins are different).
**Uno Platform Notes**: If you do not control the API server, you must either proxy the requests through a server you control, or ask the API maintainers to add CORS support. For testing, tools like "CORS Anywhere" can proxy queries.
**Reference**: https://platform.uno/docs/articles/common-issues-wasm.html

---

### AOT Debugging Limitations
**Rule**: Expect incomplete stack traces when debugging AOT-compiled WebAssembly builds.
**Why**: WebAssembly's MVP specification does not support stack walking. When running with AOT or Profiled AOT, exceptions do not provide complete .NET stack traces. Browser-native stack traces can still be viewed in the console by enabling mono trace logging.
**Example (enable mono tracing for AOT builds)**:
```xml
<PropertyGroup>
    <WasmShellRuntimeOptions>--trace E</WasmShellRuntimeOptions>
</PropertyGroup>
```
This enables error-level tracing. For more verbose output:
```xml
<WasmShellRuntimeOptions>--trace W</WasmShellRuntimeOptions> <!-- Warnings -->
<WasmShellRuntimeOptions>--trace I</WasmShellRuntimeOptions> <!-- Info -->
```
**Common Mistakes**:
- Debugging AOT builds and wondering why stack traces are missing (use interpreter mode for debugging).
- Not switching to interpreter mode for development (AOT is for release/performance profiling).
- Leaving verbose tracing enabled in production (significant performance overhead).
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/debugger-support.html

---

### Mixed Content and HTTPS Requirements
**Rule**: Ensure both the app and the debug proxy use the same protocol (HTTPS or HTTP) to avoid mixed content errors.
**Why**: When running a Wasm app over HTTPS, the browser blocks insecure WebSocket connections to the debug proxy. This manifests as a "Mixed Content" error that prevents debugging and can break runtime communication.
**Example (error)**:
```
Mixed Content: The page at 'https://localhost:5002' was loaded over HTTPS,
but attempted to connect to the insecure WebSocket endpoint 'ws://XXXX:59867/rc'.
This request has been blocked.
```
**Fix**: Ensure the development server uses consistent protocol:
```json
// launchSettings.json - use HTTPS consistently
{
  "profiles": {
    "MyApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "applicationUrl": "https://localhost:5001",
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
    }
  }
}
```
**Common Mistakes**:
- Mixing `http://` and `https://` URLs in the development setup.
- Not noticing the mixed content warning because the browser blocks it silently.
- Disabling HTTPS in development and then encountering the issue only in staging/production.
**Reference**: https://platform.uno/docs/articles/common-issues-wasm.html
