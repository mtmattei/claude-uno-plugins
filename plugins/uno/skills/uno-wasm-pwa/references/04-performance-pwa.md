# Performance and PWA Reference

Detailed reference for WebAssembly performance optimization and Progressive Web App (PWA) support in Uno Platform applications.

---

### Enable AOT Compilation for Release Builds
**Rule**: Enable `RunAOTCompilation` in your csproj for release builds to compile .NET IL to native WebAssembly.
**Why**: AOT (Ahead-of-Time) compilation converts .NET intermediate language to native WebAssembly during build, eliminating the interpreter overhead at runtime. This typically delivers 3-10x performance improvement for CPU-bound operations. The tradeoff is a larger payload size (roughly 2-3x) and significantly longer build times (5-20 minutes depending on app size).
**Example (csproj)**:
```xml
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <RunAOTCompilation>true</RunAOTCompilation>
</PropertyGroup>
```
To enable AOT on regular builds (not just publish):
```xml
<PropertyGroup>
    <RunAOTCompilationAfterBuild>true</RunAOTCompilationAfterBuild>
</PropertyGroup>
```
**Execution mode options**:
```xml
<!-- Full AOT (best performance, largest payload) -->
<WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
<RunAOTCompilation>true</RunAOTCompilation>

<!-- Jiterpreter (good balance, no build time penalty) -->
<WasmShellEnableJiterpreter>true</WasmShellEnableJiterpreter>

<!-- Interpreter only (smallest payload, slowest execution) -->
<WasmShellMonoRuntimeExecutionMode>Interpreter</WasmShellMonoRuntimeExecutionMode>
```
**Common Mistakes**:
- Expecting AOT to activate during `dotnet build` (it only activates during `dotnet publish` unless `RunAOTCompilationAfterBuild` is set).
- Not installing `wasm-tools` workload, which causes error "The MonoAOTCompiler task was not found."
- Enabling AOT in Debug configuration, which makes the edit-build-run cycle extremely slow.
- Not running on Linux/macOS for AOT builds in CI (AOT compilation is more reliable on these platforms).
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/runtime-execution-modes.html

---

### Use Profile-Guided AOT for Balanced Size and Performance
**Rule**: Generate an AOT profile from real usage patterns to selectively compile only hot methods to WebAssembly.
**Why**: Full AOT compiles everything, producing large payloads. Profile-Guided AOT only compiles methods that are actually executed during a profiling session, reducing payload size by 40-50% compared to full AOT while retaining most performance gains. The RayTracer benchmark shows uncompressed size dropping from 5.5MB to 2.9MB with this approach.
**Example (workflow)**:
```xml
<!-- Step 1: Enable profile generation -->
<PropertyGroup>
    <WasmShellGenerateAOTProfile>true</WasmShellGenerateAOTProfile>
</PropertyGroup>
```
1. Run the app without debugger (`Ctrl+F5`).
2. Navigate through the app's primary workflows.
3. Press `Alt+Shift+P` or run `App.saveProfile()` in the browser console.
4. Download the `aot.profile` file and place it next to the csproj (or in `Platforms/WebAssembly/` for Uno.Sdk projects).

```xml
<!-- Step 2: Use the profile for AOT compilation -->
<PropertyGroup>
    <!-- Remove or comment out WasmShellGenerateAOTProfile -->
    <WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
    <RunAOTCompilation>true</RunAOTCompilation>
</PropertyGroup>

<!-- If NOT using Uno.Sdk, reference the profile explicitly -->
<ItemGroup>
    <WasmShellEnableAotProfile Include="aot.profile" />
</ItemGroup>
```
**Common Mistakes**:
- Forgetting to remove `WasmShellGenerateAOTProfile` before the production build (it enables profiling mode, not AOT).
- Not re-generating the profile after major code changes (the profile is a snapshot of method usage).
- Placing the `aot.profile` in the wrong directory (for Uno.Sdk projects, use `Platforms/WebAssembly/`).
- Not using `dotnet publish` for the final build (AOT only activates during publish).
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/runtime-execution-modes.html

---

### Configure IL Trimming for Minimal Payload
**Rule**: Keep IL trimming enabled (it is on by default in Release) and use XAML Resource Trimming for additional savings.
**Why**: IL trimming removes unreachable code and unused types from the published output. XAML Resource Trimming goes further by removing XAML styles for UI controls not referenced in your app. Together, they can reduce total IL payload from ~13MB to ~9MB and `dotnet.wasm` from ~53MB to ~29MB for a default template app.
**Example (csproj)**:
```xml
<!-- IL trimming is enabled by default in Release. To be explicit: -->
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <PublishTrimmed>true</PublishTrimmed>
</PropertyGroup>

<!-- XAML Resource Trimming (opt-in, additional savings) -->
<PropertyGroup>
    <UnoXamlResourcesTrimming>true</UnoXamlResourcesTrimming>
</PropertyGroup>
```
**LinkerConfig.xml for preserving assemblies**:
```xml
<!-- Platforms/WebAssembly/LinkerConfig.xml -->
<linker>
  <!-- Preserve types used by reflection or XamlReader -->
  <assembly fullname="MyApp.Models" preserve="all" />

  <!-- Preserve specific types -->
  <assembly fullname="MyApp">
    <type fullname="MyApp.Services.DynamicService" preserve="all" />
  </assembly>
</linker>
```
**Common Mistakes**:
- Disabling trimming entirely because of a single runtime error (fix with targeted LinkerConfig entries instead).
- Using `<assembly fullname="Uno.UI" preserve="all" />` which negates all XAML trimming benefits.
- Not testing the published (trimmed) build before deployment (trimming can remove types used via reflection).
- Forgetting that `UnoXamlResourcesTrimming` must be set in all projects that reference Uno.WinUI, not just the head project.
**Reference**: https://platform.uno/docs/articles/features/resources-trimming.html

---

### Optimize Startup Time
**Rule**: Minimize initial payload and use the Jiterpreter for best startup-to-interactive time without AOT build overhead.
**Why**: WebAssembly app startup involves downloading the runtime, assemblies, and framework files, then initializing the .NET runtime. Each MB of payload adds roughly 0.5-1 second on a 4G connection. The Jiterpreter provides a JIT-like experience without the large AOT payload, making it the best option for startup-sensitive apps.
**Example (csproj - startup optimization)**:
```xml
<PropertyGroup>
    <!-- Enable Jiterpreter for runtime code generation (no build-time cost) -->
    <WasmShellEnableJiterpreter>true</WasmShellEnableJiterpreter>

    <!-- Enable XAML resource trimming to reduce assembly count -->
    <UnoXamlResourcesTrimming>true</UnoXamlResourcesTrimming>

    <!-- Load only required culture resources -->
    <!-- Remove this property to load only the user's culture (default) -->
    <!-- <WasmShellLoadAllSatelliteResources>true</WasmShellLoadAllSatelliteResources> -->
</PropertyGroup>
```
**Startup time budget breakdown (typical 4G connection)**:
| Phase | Unoptimized | Optimized |
|---|---|---|
| Download runtime (~15-50MB) | 4-12s | 2-5s (compressed) |
| Download assemblies (~5-13MB) | 1-3s | 0.5-1.5s (trimmed) |
| Runtime initialization | 1-2s | 0.5-1s (Jiterpreter) |
| First render | 0.5-1s | 0.3-0.5s |
| **Total** | **6.5-18s** | **3.3-8s** |

**Common Mistakes**:
- Loading all satellite resource assemblies when only one language is needed.
- Including debug symbols in production builds (PDB files add significant payload).
- Not enabling server-side compression (gzip/Brotli), which is the single biggest startup improvement.
- Ignoring the browser cache - second visits are near-instant with proper caching headers.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/runtime-execution-modes.html

---

### Configure the PWA Manifest
**Rule**: Add a `manifest.webmanifest` file with `Content` build action and set the `WasmPWAManifestFile` property.
**Why**: The PWA manifest tells the browser how to display the app when installed (name, icons, theme color, display mode). Without it, the browser cannot offer the "Install" prompt, and the app cannot function as a standalone PWA. The file must use the `Content` build action so it is copied to the output folder, and must NOT be placed in the `wwwroot` folder.
**Example (manifest.webmanifest)**:
```json
{
  "short_name": "MyApp",
  "name": "My Uno Platform App",
  "description": "A cross-platform app built with Uno Platform",
  "icons": [
    {
      "src": "icons/icon-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "icons/icon-512.png",
      "type": "image/png",
      "sizes": "512x512"
    },
    {
      "src": "icons/icon-1024.png",
      "type": "image/png",
      "sizes": "1024x1024",
      "purpose": "any"
    }
  ],
  "start_url": "/",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#7C3AED",
  "scope": "/"
}
```
**Example (csproj)**:
```xml
<PropertyGroup>
    <!-- Set the PWA manifest filename -->
    <WasmPWAManifestFile>manifest.webmanifest</WasmPWAManifestFile>
</PropertyGroup>

<!-- Ensure the manifest file has Content build action -->
<!-- The file should be in the project root or Platforms/WebAssembly, NOT in wwwroot -->
```
**Common Mistakes**:
- Placing the manifest file inside the `wwwroot` folder (it should be outside `wwwroot` with `Content` build action, and the bootstrapper copies it).
- Using `None` or `EmbeddedResource` build action instead of `Content`.
- Missing a 512x512 icon (Chrome requires at least 192x192 and 512x512 for the install prompt).
- Setting `"display": "browser"` (this disables standalone mode, which defeats the purpose of PWA installation).
- Using `manifest.json` extension instead of `manifest.webmanifest` without updating `WasmPWAManifestFile`.
**Uno Platform Notes**: iOS home screen icon support uses the 1024x1024 icon from the manifest. If this icon is not provided, iOS generates a scaled-down screenshot instead. Use the App Image Generator tool to create all required sizes.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-pwa.html

---

### Meet PWA Install Prompt Requirements
**Rule**: Fulfill all browser-specific PWA installability criteria to trigger the native install prompt.
**Why**: Chrome, Edge, and other browsers require specific conditions before showing the install prompt. Missing any single requirement silently prevents installation with no user-visible error. The Chrome DevTools "Application" tab's "Manifest" section shows exactly which criteria are met or missing.
**Example (PWA installability checklist)**:
```
Chrome/Edge install prompt requirements:
1. Valid manifest.webmanifest linked in index.html
2. App served over HTTPS (or localhost for development)
3. Registered service worker with a fetch handler
4. Manifest includes:
   - "name" or "short_name"
   - "start_url"
   - "display" set to "standalone", "minimal-ui", or "fullscreen"
   - "icons" with at least 192x192 and 512x512 PNG icons
   - "prefer_related_applications" is NOT set to true
5. User has interacted with the page (click, tap, or scroll)
```
**Validation in Chrome DevTools**:
1. Open DevTools (`F12`).
2. Go to **Application** tab.
3. Click **Manifest** in the sidebar.
4. Check for warnings and errors in the "Installability" section.
5. Click **Service Workers** to verify the service worker is registered and active.

**Common Mistakes**:
- Not serving over HTTPS in production (mandatory for service worker registration).
- Having an empty or invalid service worker (it must include a `fetch` event handler).
- Testing the install prompt without user interaction (Chrome requires at least one engagement event).
- Not checking the Application tab for specific installability failures.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-pwa.html

---

### Configure the Service Worker
**Rule**: Register a service worker with appropriate caching strategies for offline support and faster repeat loads.
**Why**: A service worker intercepts network requests and can serve cached responses, enabling offline functionality and near-instant repeat loads. For Uno Platform WebAssembly apps, the service worker should cache the `/_framework/` assets aggressively (they are immutable) while using a network-first strategy for `index.html` and `blazor.boot.json` to ensure updates are detected.
**Example (service-worker.js)**:
```javascript
const CACHE_NAME = 'myapp-v1';
const IMMUTABLE_CACHE = 'myapp-immutable';

// Assets that change with each deployment
const MUTABLE_ASSETS = [
    '/',
    '/index.html'
];

self.addEventListener('install', event => {
    event.waitUntil(
        caches.open(CACHE_NAME).then(cache => cache.addAll(MUTABLE_ASSETS))
    );
    self.skipWaiting();
});

self.addEventListener('activate', event => {
    event.waitUntil(
        caches.keys().then(keys =>
            Promise.all(
                keys.filter(key => key !== CACHE_NAME && key !== IMMUTABLE_CACHE)
                    .map(key => caches.delete(key))
            )
        )
    );
    self.clients.claim();
});

self.addEventListener('fetch', event => {
    const url = new URL(event.request.url);

    // Framework assets are immutable - cache forever
    if (url.pathname.startsWith('/_framework/') &&
        !url.pathname.endsWith('blazor.boot.json') &&
        !url.pathname.endsWith('dotnet.js')) {
        event.respondWith(
            caches.open(IMMUTABLE_CACHE).then(cache =>
                cache.match(event.request).then(cached =>
                    cached || fetch(event.request).then(response => {
                        cache.put(event.request, response.clone());
                        return response;
                    })
                )
            )
        );
        return;
    }

    // Everything else: network-first with cache fallback
    event.respondWith(
        fetch(event.request)
            .then(response => {
                const clone = response.clone();
                caches.open(CACHE_NAME).then(cache =>
                    cache.put(event.request, clone)
                );
                return response;
            })
            .catch(() => caches.match(event.request))
    );
});
```
**Common Mistakes**:
- Caching `blazor.boot.json` with the immutable strategy (prevents app updates from being detected).
- Not versioning the cache name, which prevents old caches from being cleaned up.
- Forgetting `self.skipWaiting()` and `self.clients.claim()`, which delays activation.
- Not handling the `fetch` event at all (required for PWA installability).
- Caching API responses that should always be fresh.
**Reference**: https://platform.uno/docs/articles/external/uno.wasm.bootstrap/doc/features-pwa.html

---

### Reduce Payload Size Strategically
**Rule**: Apply payload reduction techniques in order of impact: compression, trimming, XAML trimming, then Profiled AOT.
**Why**: Each technique has different effort-to-impact ratios. Server compression is the easiest and most impactful, while Profiled AOT requires the most setup. Applying them in order ensures maximum benefit with minimum effort.
**Example (impact summary for a default Uno Platform template app)**:
| Technique | Effort | Payload Reduction | Build Time Impact |
|---|---|---|---|
| Brotli compression (server) | Low | 60-70% transfer reduction | None |
| IL trimming (default in Release) | None | ~10-15% IL reduction | Minimal |
| XAML Resource Trimming | Low | ~30% IL, ~45% dotnet.wasm | Minimal |
| Jiterpreter (no full AOT) | Low | No increase (avoids AOT bloat) | None |
| Profiled AOT | Medium | Selective: smaller than full AOT | +5-10 min |
| Full AOT | Low (config) | 2-3x size INCREASE for perf | +10-20 min |

**Example (recommended production configuration)**:
```xml
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <!-- Trimming (on by default, be explicit) -->
    <PublishTrimmed>true</PublishTrimmed>
    <UnoXamlResourcesTrimming>true</UnoXamlResourcesTrimming>

    <!-- Jiterpreter for good perf without AOT payload cost -->
    <WasmShellEnableJiterpreter>true</WasmShellEnableJiterpreter>

    <!-- Only load needed cultures -->
    <!-- WasmShellLoadAllSatelliteResources defaults to false -->
</PropertyGroup>
```
**Common Mistakes**:
- Enabling full AOT and wondering why the payload doubled (AOT trades size for speed).
- Ignoring server compression, which provides the largest single improvement.
- Assuming that "smaller IL" automatically means "faster app" (execution mode matters more than payload size for runtime performance).
- Not measuring actual transfer sizes (use Chrome DevTools Network tab with "Disable cache" enabled).
**Reference**: https://platform.uno/docs/articles/features/resources-trimming.html
