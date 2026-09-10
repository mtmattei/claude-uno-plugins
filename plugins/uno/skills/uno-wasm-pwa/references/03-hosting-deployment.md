# Hosting and Deployment Reference

Detailed reference for hosting and deploying Uno Platform WebAssembly applications on various platforms, including caching strategies, compression, and MIME type configuration.

---

### Use dotnet serve for Local Testing
**Rule**: Use `dotnet serve` to test published WebAssembly output locally before deploying to a server.
**Why**: The `dotnet serve` tool provides a lightweight static file server that correctly handles WebAssembly content types. Testing locally catches missing files, MIME type issues, and routing problems before they reach production. The published output must be served from the `wwwroot` subfolder.
**Example (CLI)**:
```bash
# Install dotnet-serve (one time)
dotnet tool install -g dotnet-serve

# Publish the app
dotnet publish -f net9.0-browserwasm -c Release -o ./publish

# Serve the published output
dotnet serve -d ./publish/wwwroot -p 8000

# App is now available at http://localhost:8000
```
For debug builds:
```bash
dotnet publish -f net9.0-browserwasm -c Debug
dotnet serve -d bin/Debug/net9.0-browserwasm/publish/wwwroot -p 8000
```
**Common Mistakes**:
- Serving the `publish` folder instead of `publish/wwwroot` (the app files are inside `wwwroot`).
- Using `dotnet run` for local testing instead of publishing first (the output structure differs).
- Forgetting to publish before serving, which means stale files are served.
**Reference**: https://platform.uno/docs/articles/uno-publishing-webassembly.html

---

### Deploy to Azure Static Web Apps
**Rule**: Use Azure Static Web Apps with a `staticwebapp.config.json` file for deep linking and proper caching headers.
**Why**: Azure Static Web Apps is purpose-built for static sites and integrates with GitHub Actions and Azure DevOps for CI/CD. The `staticwebapp.config.json` file controls routing fallbacks (needed for SPA deep linking) and cache headers for framework assets.
**Example (wwwroot/staticwebapp.config.json)**:
```json
{
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/package_*"]
  },
  "routes": [
    {
      "route": "/_framework/blazor.boot.*",
      "headers": {
        "cache-control": "must-revalidate, max-age=3600"
      }
    },
    {
      "route": "/_framework/dotnet.js",
      "headers": {
        "cache-control": "must-revalidate, max-age=3600"
      }
    },
    {
      "route": "/_framework/*",
      "headers": {
        "cache-control": "public, immutable, max-age=31536000"
      }
    },
    {
      "route": "/package_*",
      "headers": {
        "cache-control": "public, immutable, max-age=31536000"
      }
    },
    {
      "route": "/*",
      "headers": {
        "cache-control": "must-revalidate, max-age=3600"
      }
    }
  ]
}
```
**Example (GitHub Actions workflow)**:
```yaml
name: Deploy to Azure Static Web Apps

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches: [main]

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'

      - name: Install wasm-tools
        run: dotnet workload install wasm-tools

      - name: Publish
        run: dotnet publish -f net9.0-browserwasm -c Release -o ./publish

      - name: Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          action: "upload"
          app_location: "./publish/wwwroot"
          skip_app_build: true
```
**Common Mistakes**:
- Not placing `staticwebapp.config.json` in the `wwwroot` folder (it must be at the root of the served content).
- Forgetting the `navigationFallback` section, which causes 404 errors on deep links and page refreshes.
- Setting `app_location` to `./publish` instead of `./publish/wwwroot`.
- Not installing `wasm-tools` workload in the CI pipeline.
**Reference**: https://platform.uno/docs/articles/guides/azure-static-webapps.html

---

### Configure Nginx MIME Types and Compression
**Rule**: Add `application/wasm` MIME type and enable gzip compression for `.wasm` files in Nginx.
**Why**: Without the correct MIME type, browsers refuse to compile `.wasm` files. Missing compression means full-size downloads for .wasm files that are typically 20-50MB uncompressed. Proper configuration reduces initial load time by 60-80%.
**Example (nginx.conf)**:
```nginx
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Add WebAssembly MIME types
    types {
        application/wasm wasm;
        application/octet-stream clr;
        application/octet-stream pdb;
        application/font-woff woff;
        application/font-woff woff2;
    }

    # Enable gzip for Wasm content
    gzip on;
    gzip_min_length 10240;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css text/xml text/javascript
               application/x-javascript application/json
               application/xml application/wasm application/octet-stream;

    server {
        listen 80;

        # SPA fallback for deep linking
        location ~ ^\/(?!(package_)) {
            try_files $uri $uri/ /index.html;
            add_header Cache-Control "must-revalidate, max-age=3600";
        }

        # Immutable framework assets
        location ~ ^\/package_ {
            try_files $uri $uri/ =404;
            add_header Cache-Control "public, immutable, max-age=31536000";
        }
    }
}
```
**Common Mistakes**:
- Forgetting the `application/wasm wasm;` MIME type, which causes "CompileError: WebAssembly.instantiate()" failures.
- Not including `application/wasm` in the `gzip_types` list.
- Placing the `types` block outside the `http` block.
- Not restarting Nginx after configuration changes (`nginx -s reload`).
**Reference**: https://platform.uno/docs/articles/how-to-host-a-webassembly-app.html

---

### Enable Brotli Compression in Nginx
**Rule**: Enable Brotli compression for WebAssembly apps when the `ngx-brotli` module is available.
**Why**: Brotli achieves 15-25% better compression ratios than gzip for WebAssembly binaries. For a typical Uno Platform app with a 20MB uncompressed payload, Brotli can reduce transfer size by an additional 2-4MB compared to gzip alone.
**Example (nginx.conf - requires ngx-brotli module)**:
```nginx
# Serve pre-compressed .br files if they exist
brotli_static on;

# Dynamic Brotli compression for text-based content
brotli on;
brotli_types text/plain text/css application/json
             application/javascript application/x-javascript
             text/javascript;
brotli_comp_level 4;
```
**Common Mistakes**:
- Assuming Brotli is built into Nginx (it requires the separate `ngx-brotli` module to be compiled and loaded).
- Setting `brotli_comp_level` too high (6+) for dynamic compression, which adds latency. Use 4 for dynamic, and pre-compress static assets at level 11.
- Not including `brotli_static on;` to serve pre-compressed `.br` files that the .NET SDK generates during publish.
**Uno Platform Notes**: The .NET SDK handles Brotli pre-compression during `dotnet publish`. Ensure your server is configured to serve the `.br` files when the `Accept-Encoding: br` header is present.
**Reference**: https://platform.uno/docs/articles/how-to-host-a-webassembly-app.html

---

### Configure Apache for WebAssembly
**Rule**: Add WebAssembly MIME types using `AddType` directives in Apache configuration.
**Why**: Apache does not include MIME type mappings for `.wasm`, `.clr`, or `.pdb` files by default. Without these, the browser receives incorrect content types and fails to load the app.
**Example (httpd.conf or apache2.conf)**:
```apache
# Add to the root configuration file
AddType application/wasm .wasm
AddType application/octet-stream .clr
AddType application/octet-stream .dat
AddType application/octet-stream .pdb
AddType application/font-woff .woff
AddType application/font-woff .woff2

# Enable SPA fallback for deep linking
<Directory "/var/www/html">
    FallbackResource /index.html
</Directory>

# Enable compression
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE application/wasm
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/json
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/html
</IfModule>

# Cache immutable framework assets
<IfModule mod_headers.c>
    <FilesMatch "\.(wasm|dll)$">
        Header set Cache-Control "public, immutable, max-age=31536000"
    </FilesMatch>
</IfModule>
```
**Common Mistakes**:
- Editing a virtual host file instead of the main config, causing MIME types to apply only to one site.
- Forgetting to restart Apache after changes (`systemctl restart apache2`).
- Not enabling `mod_deflate` or `mod_headers` modules (`a2enmod deflate headers`).
**Reference**: https://platform.uno/docs/articles/how-to-host-a-webassembly-app.html

---

### Deploy to GitHub Pages
**Rule**: Use a GitHub Actions workflow to publish and deploy to GitHub Pages with a proper base path.
**Why**: GitHub Pages serves content from a subpath (e.g., `/repo-name/`), which requires the app's `<base href>` to match. Without this, all relative asset references break and the app fails to load.
**Example (GitHub Actions workflow)**:
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'

      - name: Install wasm-tools
        run: dotnet workload install wasm-tools

      - name: Publish
        run: dotnet publish -f net9.0-browserwasm -c Release -o ./publish

      - name: Fix base path
        run: sed -i 's|<base href="/" />|<base href="/my-repo-name/" />|g' ./publish/wwwroot/index.html

      - name: Add .nojekyll
        run: touch ./publish/wwwroot/.nojekyll

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./publish/wwwroot

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
**Common Mistakes**:
- Not updating `<base href>` to match the repository name subpath.
- Forgetting the `.nojekyll` file (GitHub Pages runs Jekyll by default, which ignores files starting with `_` like `_framework/`).
- Using the `gh-pages` branch approach without the `.nojekyll` file.
- Not setting the correct permissions in the workflow file for GitHub Pages deployment.
**Reference**: https://platform.uno/docs/articles/uno-publishing-webassembly.html

---

### Implement Proper Caching Strategy for Framework Assets
**Rule**: Cache `/_framework/` assets as immutable with a one-year max-age, but force revalidation for `blazor.boot.json`, `dotnet.js`, and `index.html`.
**Why**: Framework assets under `/_framework/` (excluding the boot manifest and runtime loader) are content-addressed and immutable - their filenames change when content changes. Caching them aggressively eliminates re-downloads on repeat visits. However, `blazor.boot.json`, `dotnet.js`, and `index.html` must always be revalidated because they reference the current set of framework files.
**Example (caching rules summary)**:

| Path Pattern | Cache-Control | Rationale |
|---|---|---|
| `/_framework/*` (general) | `public, immutable, max-age=31536000` | Content-addressed filenames |
| `/_framework/blazor.boot.json` | `must-revalidate, max-age=3600` | Boot manifest changes per release |
| `/_framework/dotnet.js` | `must-revalidate, max-age=3600` | Runtime loader changes per release |
| `/index.html` | `must-revalidate, max-age=3600` | Entry point references current assets |
| `/package_*` | `public, immutable, max-age=31536000` | Immutable packaged assets |
| All other root files | `must-revalidate, max-age=3600` | May change between deployments |

**Common Mistakes**:
- Applying `immutable, max-age=31536000` to `index.html` or `blazor.boot.json`, which prevents users from getting app updates.
- Not caching `/_framework/` assets at all, causing full re-downloads on every page load (10-50MB per visit).
- Using `no-cache` instead of `must-revalidate` (subtle difference: `no-cache` always revalidates, while `must-revalidate` uses the asset until `max-age` expires).
**Reference**: https://platform.uno/docs/articles/how-to-host-a-webassembly-app.html
