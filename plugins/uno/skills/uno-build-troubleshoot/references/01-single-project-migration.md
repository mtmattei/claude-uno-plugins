# Single Project Migration Reference

Detailed patterns for migrating Uno Platform projects from the legacy multi-project structure to the Uno.Sdk Single Project model.

---

### Use a Reference Project as Your Migration Blueprint

**Rule**: Always scaffold a fresh `dotnet new unoapp` project with the same name before migrating; never build a Single Project csproj from scratch.

**Why**: The generated project contains the correct Uno.Sdk imports, TFM list, Platforms/ folder layout, and App.xaml boilerplate. Hand-authoring these from documentation is error-prone and misses defaults that the template embeds (e.g., implicit usings, Uno feature flags). Using a reference project reduces migration time from hours to minutes and eliminates an entire class of structural errors.

**Example (CLI)**:
```bash
# Create a reference project with the recommended preset
dotnet new unoapp -o MyApp --preset=recommended

# The generated structure you will use as your blueprint:
# MyApp/
#   global.json
#   MyApp.sln
#   MyApp/
#     MyApp.csproj
#     App.xaml / App.xaml.cs
#     Platforms/
#       Android/
#       iOS/
#       Desktop/
#       WebAssembly/
```

**Common Mistakes**:
- Writing the csproj from scratch by copying XML snippets from blog posts or outdated guides
- Using `dotnet new unoapp` with a different project name than the original, causing namespace mismatches when merging code
- Forgetting to pass `--preset=recommended` and getting a bare template missing Uno.Extensions, Toolkit, or Navigation

**Uno Platform Notes**: Migration to Single Project is optional. Existing multi-project solutions continue to work. Migrate when you want simplified maintenance, unified NuGet management, and access to newer Uno.Sdk features.

**Reference**: https://platform.uno/docs/articles/howto-migrate-existing-code.html

---

### Configure global.json with the Uno.Sdk Version

**Rule**: Place a `global.json` file next to the solution file with the `Uno.Sdk` entry under `msbuild-sdks`; never reference the Uno SDK via a NuGet `<Sdk>` element in the csproj.

**Why**: The Uno.Sdk is an MSBuild SDK that must be resolved before project evaluation begins. The `global.json` mechanism is the only reliable way to pin the SDK version across all projects in the solution. Without it, builds fail with cryptic "SDK not found" errors or silently use a cached/wrong version.

**Example (global.json)**:
```json
{
    "msbuild-sdks": {
        "Uno.Sdk": "5.6.22"
    }
}
```

**Example (csproj - first line)**:
```xml
<Project Sdk="Uno.Sdk">
    <!-- The version comes from global.json, NOT here -->
</Project>
```

**Common Mistakes**:
- Placing `global.json` inside the project folder instead of next to the `.sln` file
- Adding a version attribute on the Sdk element: `<Project Sdk="Uno.Sdk/5.6.22">` - this conflicts with global.json and causes version resolution failures
- Forgetting to update `global.json` when upgrading the Uno.Sdk, leading to stale SDK versions
- Nesting `global.json` inside a subdirectory where MSBuild cannot discover it during solution-level builds

**Uno Platform Notes**: If you have multiple solutions in a monorepo, place `global.json` at the repo root. MSBuild walks up the directory tree to find it. Each solution can have its own `global.json` if different SDK versions are required, but this is discouraged.

**Reference**: https://platform.uno/docs/articles/external/uno.sdk/doc/articles/overview.html

---

### Order Target Frameworks Correctly to Avoid UNOB0011/UNOB0013

**Rule**: Never place `net9.0` (the base/library TFM) as the first entry in `<TargetFrameworks>`; always list a platform-specific TFM first.

**Why**: Visual Studio 2022 uses the first TFM in the list to drive IntelliSense, code completion, and the XAML designer context. When `net9.0` is first, VS2022 resolves against the base class library where Uno Platform APIs are not available, producing false UNOB0011 ("Project is not a valid Uno Platform project") and UNOB0013 errors. The build itself may also fail because the SDK interprets the project type from the first TFM.

**Example (csproj - correct)**:
```xml
<PropertyGroup>
    <TargetFrameworks>net9.0-android;net9.0-ios;net9.0-maccatalyst;net9.0-browserwasm;net9.0-desktop;net9.0</TargetFrameworks>
</PropertyGroup>
```

**Example (csproj - WRONG)**:
```xml
<!-- DO NOT do this -->
<PropertyGroup>
    <TargetFrameworks>net9.0;net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop</TargetFrameworks>
</PropertyGroup>
```

**Common Mistakes**:
- Alphabetically sorting TFMs, which puts `net9.0` before `net9.0-android`
- Removing all platform TFMs during debugging and leaving only `net9.0`
- Adding `net9.0` first when creating a class library that needs to also target the base framework

**Uno Platform Notes**: The `net9.0` TFM at the end is used for shared class libraries and unit test projects that reference the Uno Platform app. It must be present if the project is consumed as a library, but it must always be last.

**Reference**: https://platform.uno/docs/articles/troubleshooting-build-errors.html

---

### Consolidate Platform Code into the Platforms/ Folder

**Rule**: Move all platform-specific files from legacy per-platform projects into the corresponding `Platforms/{PlatformName}/` subdirectory within the Single Project, preserving the original file structure under each platform folder.

**Why**: The Single Project convention uses `Platforms/` subfolders to isolate code that compiles only for a specific TFM. The Uno.Sdk automatically applies the correct `<Compile>` includes based on folder names. Placing Android manifests, iOS entitlements, or Wasm `index.html` files outside the correct Platforms/ subfolder causes them to be included in all TFMs, producing compile errors or resource conflicts.

**Example (folder mapping)**:
```
Legacy multi-project:              Single Project:
MyApp.Wasm/                   -->  MyApp/Platforms/WebAssembly/
MyApp.Wasm/wwwroot/           -->  MyApp/Platforms/WebAssembly/wwwroot/
MyApp.Mobile/Android/         -->  MyApp/Platforms/Android/
MyApp.Mobile/iOS/             -->  MyApp/Platforms/iOS/
MyApp.Skia.Gtk/               -->  MyApp/Platforms/Desktop/
MyApp.Skia.WPF/               -->  (merged into Desktop or removed)
MyApp.Windows/                -->  MyApp/Platforms/Windows/
```

**Example (resulting structure)**:
```
MyApp/
  Platforms/
    Android/
      AndroidManifest.xml
      MainActivity.cs
      Resources/
    iOS/
      Entitlements.plist
      Info.plist
      LaunchScreen.storyboard
    Desktop/
      Program.cs
    WebAssembly/
      wwwroot/
      Program.cs
    Windows/
      Package.appxmanifest
```

**Common Mistakes**:
- Leaving platform files in the project root, causing them to compile for all targets
- Forgetting to move `AndroidManifest.xml` into `Platforms/Android/`, resulting in missing permissions at runtime
- Duplicating `Program.cs` across platforms without adjusting entry points for each target
- Not removing the old platform project directories after migration, causing solution confusion

**Uno Platform Notes**: The Skia.Gtk and Skia.WPF host projects from legacy solutions both map to `Platforms/Desktop/` in Single Project. The Uno.Sdk Desktop target uses the Skia renderer by default and does not require separate GTK or WPF host projects.

**Reference**: https://platform.uno/docs/articles/howto-migrate-existing-code.html

---

### Merge App.xaml and App.xaml.cs from the Reference Project

**Rule**: Copy `App.xaml` and `App.xaml.cs` from the reference project first, then merge your custom startup logic (DI registration, route configuration, logging) into the reference structure - never the other way around.

**Why**: The Single Project `App.xaml.cs` has a fundamentally different initialization sequence than legacy multi-project apps. It uses `this.Build()` with a fluent host builder instead of manually wiring up platform bootstrappers. Trying to retrofit the new initialization into your old `App.xaml.cs` almost always produces subtle startup failures because the builder call order matters.

**Example (App.xaml.cs - correct merge pattern)**:
```csharp
public partial class App : Application
{
    protected override void OnLaunched(LaunchActivatedEventArgs args)
    {
        var builder = this.CreateBuilder(args)
            .Configure(host => host
                // Your custom DI registrations merged here
                .UseConfiguration(configure: configBuilder =>
                    configBuilder.EmbeddedSource<App>())
                .UseLocalization()
                .UseHttp()
                .ConfigureServices((context, services) =>
                {
                    // Merge your service registrations from old App.xaml.cs
                    services.AddSingleton<IMyService, MyService>();
                })
                .UseNavigation(ReactiveViewModelMapping.ViewModelMappings,
                    RegisterRoutes)
            );

        MainWindow = builder.Window;
        Host = await builder.NavigateAsync<Shell>();
    }

    // Merge your route registrations
    private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
    {
        views.Register(
            new ViewMap<MainPage, MainViewModel>()
        );
        routes.Register(
            new RouteMap("Main", View: views.FindByViewModel<MainViewModel>())
        );
    }
}
```

**Common Mistakes**:
- Starting from the old `App.xaml.cs` and trying to add `this.Build()` calls into the legacy structure
- Losing the `RegisterRoutes` method during the merge, breaking all navigation
- Not replacing `AppHead` references with `App` in code-behind files
- Forgetting to merge custom `InitializeLogging()` calls, losing diagnostic output
- Overwriting `App.xaml` resource dictionary merges that pull in Material/Toolkit themes

**Uno Platform Notes**: In legacy projects, the entry point class was often named `AppHead` (in the platform head project). Single Project uses `App` as the unified entry point. Search your entire solution for `AppHead` references and replace them with `App`.

**Reference**: https://platform.uno/docs/articles/howto-migrate-existing-code.html

---

### Use Choose Blocks for Platform-Specific MSBuild Configuration

**Rule**: Wrap platform-specific MSBuild properties and items inside `<Choose>` / `<When>` blocks that condition on `$(TargetFramework)` rather than using `Condition` attributes on individual properties scattered throughout the csproj.

**Why**: `<Choose>` blocks create a clear, readable, and maintainable structure for conditional configuration. Scattering `Condition` attributes across dozens of property groups makes the csproj difficult to audit and easy to misconfigure. Choose blocks also prevent accidental property application to the wrong TFM, which can cause silent build failures.

**Example (csproj)**:
```xml
<Choose>
    <When Condition="'$(TargetFramework)'=='net9.0-browserwasm'">
        <PropertyGroup>
            <WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
        </PropertyGroup>
    </When>
    <When Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">
        <PropertyGroup>
            <JavaMaximumHeapSize>1g</JavaMaximumHeapSize>
            <AndroidUseAapt2>true</AndroidUseAapt2>
        </PropertyGroup>
    </When>
    <When Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">
        <PropertyGroup>
            <MtouchLink>SdkOnly</MtouchLink>
        </PropertyGroup>
        <ItemGroup>
            <BundleResource Include="Platforms\iOS\Resources\**" />
        </ItemGroup>
    </When>
</Choose>
```

**Common Mistakes**:
- Using string `Contains` checks instead of proper TFM identifier functions: `Condition="$(TargetFramework.Contains('android'))"` can match unintended TFMs
- Placing Choose blocks before the `<Project Sdk="Uno.Sdk">` opening tag (invalid XML position)
- Defining the same property in both a Choose block and an unconditional PropertyGroup, causing unpredictable override behavior
- Forgetting the `Otherwise` block when a default value is needed for non-platform TFMs

**Uno Platform Notes**: The `$([MSBuild]::GetTargetPlatformIdentifier(...))` function is the recommended way to detect platforms, as it handles both the full TFM string and variations like `net9.0-ios` vs `net9.0-iossimulator`. For WebAssembly, direct string comparison against `net9.0-browserwasm` is acceptable since there is only one Wasm TFM.

**Reference**: https://learn.microsoft.com/en-us/visualstudio/msbuild/choose-element-msbuild

---

### Delete Old Platform Projects and Clean the Solution

**Rule**: After verifying the migrated Single Project builds and runs on all target platforms, remove all legacy platform-specific project directories and their references from the solution file.

**Why**: Leaving old platform projects in the solution causes confusion for new team members, creates false IntelliSense results from stale code, and risks accidentally building or deploying from the wrong project. CI/CD pipelines may also pick up old project files and produce duplicate or conflicting artifacts.

**Example (cleanup steps)**:
```bash
# 1. Verify the Single Project builds for all targets
dotnet build MyApp.csproj -f net9.0-android
dotnet build MyApp.csproj -f net9.0-browserwasm
dotnet build MyApp.csproj -f net9.0-desktop

# 2. Remove old projects from solution
dotnet sln MyApp.sln remove MyApp.Wasm/MyApp.Wasm.csproj
dotnet sln MyApp.sln remove MyApp.Mobile/MyApp.Mobile.csproj
dotnet sln MyApp.sln remove MyApp.Skia.Gtk/MyApp.Skia.Gtk.csproj
dotnet sln MyApp.sln remove MyApp.Skia.WPF/MyApp.Skia.WPF.csproj
dotnet sln MyApp.sln remove MyApp.Windows/MyApp.Windows.csproj

# 3. Delete old directories (after committing your migration in version control)
# Only do this after confirming everything works
```

**Common Mistakes**:
- Deleting old projects before verifying the Single Project builds successfully on all platforms
- Not committing the migration to version control before deleting old projects (no rollback path)
- Leaving orphaned project references in the `.sln` file that point to deleted directories
- Forgetting to update CI/CD scripts that reference old project paths or project-specific build targets

**Uno Platform Notes**: If your solution contains shared class libraries (e.g., `MyApp.Core`, `MyApp.Services`), these typically remain as separate projects. Only the platform head projects (`MyApp.Wasm`, `MyApp.Mobile`, `MyApp.Skia.*`, `MyApp.Windows`) are consolidated into the Single Project.

**Reference**: https://platform.uno/docs/articles/howto-migrate-existing-code.html

---

### Migration Is Optional - Evaluate Before Committing

**Rule**: Assess whether migration to Single Project provides meaningful value for your team before starting; do not migrate solely because the new template uses it.

**Why**: Single Project migration involves non-trivial effort (csproj rewrite, folder restructuring, App.xaml merge, CI/CD updates). For teams with stable multi-project setups, working CI pipelines, and no plans to adopt newer Uno.Sdk features, the disruption may outweigh the benefits. The legacy multi-project structure continues to be supported and receives updates.

**When to migrate**:
- You are upgrading to a new major .NET version (good time to consolidate)
- You want unified NuGet package management across platforms
- Your CI/CD is already being reworked
- You need features only available in Uno.Sdk (e.g., UnoFeatures property)

**When to defer**:
- Your multi-project setup is stable and team velocity is high
- You have complex per-platform build configurations that are hard to express in Choose blocks
- You are in the middle of a feature release cycle with tight deadlines

**Common Mistakes**:
- Starting migration without team buy-in or dedicated sprint time
- Migrating during a release freeze when the ability to hotfix is critical
- Underestimating the effort needed to update CI/CD pipelines (Docker images, GitHub Actions, Azure DevOps)
- Assuming all NuGet packages are compatible with the Single Project structure without testing

**Reference**: https://platform.uno/docs/articles/howto-migrate-existing-code.html
