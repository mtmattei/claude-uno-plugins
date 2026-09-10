---
name: uno-wasm-pwa
description: "Uno Platform WebAssembly and Progressive Web App development: bootstrapper setup, PWA manifests, debugging, hosting, performance optimization, and deployment. Use when: (1) Targeting WebAssembly with Uno Platform, (2) Adding PWA support with service workers, (3) Debugging Wasm apps, (4) Deploying to Azure Static Web Apps or Nginx, (5) Optimizing Wasm payload size and load time, (6) Configuring AOT compilation for Wasm. Do NOT use for: general project setup (see uno-platform-agent) or .NET version upgrades (see uno-build-troubleshoot)."
license: "Apache 2.0 (patterns derived from Uno Platform documentation)"
metadata:
  version: "1.0.0"
---

# Uno Platform WebAssembly and PWA

Patterns for building, debugging, optimizing, and deploying Uno Platform apps as WebAssembly applications and Progressive Web Apps.

## Prerequisites

```bash
dotnet workload install wasm-tools
```

Verify: `dotnet workload list` should show `wasm-tools` installed.

## Project Setup

The `net9.0-browserwasm` TFM is included by default in Uno Platform Single Project:

```xml
<TargetFrameworks>
    net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop;net9.0
</TargetFrameworks>
```

### Enable PWA

Add via template or manually:

```bash
dotnet new unoapp -pwa
```

Or add to an existing project by placing `manifest.webmanifest` (with `Content` build action) in the WebAssembly platform folder and registering a service worker.

## Bootstrapper (Uno.Wasm.Bootstrap 9.x)

Version 9.x targets .NET 9 and uses `Microsoft.NET.Sdk.WebAssembly`. Key changes from 8.x:

- SDK changed from `Microsoft.NET.Sdk.Web` to `Microsoft.NET.Sdk.WebAssembly`
- `Uno.Wasm.Bootstrap.DevServer` package is no longer needed
- Compression handled by .NET SDK (not bootstrapper properties)
- IDBFS not enabled by default; must be explicitly restored

### Bootstrapper Version Mapping

| Bootstrapper | .NET Runtime |
|-------------|-------------|
| 9.x | .NET 9 |
| 10.x | .NET 10 |
| 8.x | .NET 8 |

## Debugging

### Visual Studio 2022

Ensure `launchSettings.json` contains the inspect URI in every profile:

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

### Browser DevTools

- Use browser F12 console for `Console.WriteLine` output
- Source maps available in debug builds
- Set breakpoints in browser Sources tab under `file://` paths

### Common Wasm Debug Issues

- **Blank screen**: Check browser console for .NET runtime errors
- **Missing assets**: Verify `Content` build action on static files
- **CORS errors**: Configure proper headers on hosting server

## Hosting and Deployment

### Azure Static Web Apps

Publish with `dotnet publish -c Release -f net9.0-browserwasm`, then deploy the `publish/wwwroot` folder. For full CI/CD workflows (GitHub Actions, Azure Pipelines), see [references/03-hosting-deployment.md](references/03-hosting-deployment.md).

### Nginx

Required: `application/wasm` MIME type, gzip compression for `.wasm`/`.js`/`.json`, and immutable cache headers for `/_framework/` assets. For full Nginx config, see [references/03-hosting-deployment.md](references/03-hosting-deployment.md).

### Local Testing

```bash
dotnet tool install -g dotnet-serve
dotnet publish -c Debug -f net9.0-browserwasm
dotnet serve -d bin/Debug/net9.0-browserwasm/publish/wwwroot -p 8000
```

## Performance Optimization

### AOT Compilation

Ahead-of-Time compilation improves runtime performance at the cost of larger payloads and longer builds.

```xml
<PropertyGroup Condition="'$(TargetFramework)' == 'net9.0-browserwasm'">
    <RunAOTCompilation>true</RunAOTCompilation>
</PropertyGroup>
```

### Payload Size Reduction

- **Enable trimming** (default in Release): removes unused code
- **Use Brotli compression** on the web server
- **Lazy-load assemblies** for large features not needed at startup
- **Minimize embedded resources**: large images and assets inflate the initial download

### Startup Optimization

- Keep initial visual tree minimal (use `x:Load` for deferred content)
- Defer non-critical service registration
- Use splash screen during .NET runtime initialization

## Threading

WebAssembly threading is **not supported** in Uno Platform on .NET 9. Microsoft's runtime changes prevent managed code from running on the main JavaScript thread. Support may return in future .NET 10 previews.

**Workarounds**:
- Use `Task.Run` for CPU-bound work (runs on thread pool when available)
- Keep UI thread responsive with async/await
- Avoid synchronous blocking calls

## PWA Features

### Service Worker

Configured automatically when PWA is enabled. Handles offline caching of app shell and assets.

### Manifest

Standard PWA manifest with `name`, `short_name`, `start_url`, `display: standalone`, `theme_color`, and icon entries (192px + 512px). For a complete manifest template, see [references/04-performance-pwa.md](references/04-performance-pwa.md).

### Install Prompt

PWA install is handled by the browser. Ensure manifest and service worker are correctly configured for the browser to show the install prompt.

## Common Mistakes

- Forgetting `dotnet workload install wasm-tools` before building
- Using `Microsoft.NET.Sdk.Web` instead of `Microsoft.NET.Sdk.WebAssembly` with bootstrapper 9.x
- Not setting proper MIME types on hosting server (especially `application/wasm`)
- Expecting threading to work (not supported on .NET 9 Wasm)
- Not adding `inspectUri` to `launchSettings.json` for debugging
- Using deprecated bootstrapper properties from 8.x (see migration list in docs)

## Related Skills

| Skill | Use instead when... |
|-------|-------------------|
| `uno-build-troubleshoot` | Upgrading bootstrapper versions or fixing Wasm build errors |
| `uno-platform-agent` | General project setup, MVVM/MVUX, or multi-platform targeting |
| `winui-xaml` | Async/threading best practices or XAML performance optimization |
| `uno-toolkit` | Using responsive breakpoints with ResponsiveExtension |

## Detailed References

- [references/01-setup-bootstrapper.md](references/01-setup-bootstrapper.md) - Read when setting up Wasm project or upgrading bootstrapper
- [references/02-debugging.md](references/02-debugging.md) - Read when debugging Wasm apps in VS2022 or browser
- [references/03-hosting-deployment.md](references/03-hosting-deployment.md) - Read when deploying to Azure, Nginx, Apache, or GitHub Pages
- [references/04-performance-pwa.md](references/04-performance-pwa.md) - Read when optimizing payload, startup, or adding PWA features
