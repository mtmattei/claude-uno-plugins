# Project Setup & Architecture (Critical Priority)

## Single Project Structure with Uno.Sdk

**When to apply:** All new Uno Platform projects (5.2+)

### Pattern
```xml
<Project Sdk="Uno.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net9.0-android;net9.0-ios;net9.0-browserwasm;net9.0-desktop</TargetFrameworks>
    <OutputType>Exe</OutputType>
    <UnoSingleProject>true</UnoSingleProject>
  </PropertyGroup>
</Project>
```

### Key Principles
- Use `Uno.Sdk` for all new projects - abstracts cross-platform complexity
- `UnoSingleProject=true` enables single project architecture
- Order of TargetFrameworks matters: net9.0-desktop should NOT be first (VS2022 debugging issue)
- Always include net9.0 (or net8.0) as base TFM at the end

### Platform-Specific Code Organization
```
ProjectName/
├── Platforms/
│   ├── Android/
│   ├── iOS/
│   ├── Desktop/
│   └── WebAssembly/
```

### References
- Official docs: https://platform.uno/docs/articles/migrating-to-single-project.html
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\UnoApp3.csproj

## UnoFeatures Configuration

**When to apply:** Feature enablement in .csproj

### By Application Type

**Minimal App (Data Display):**
```xml
<UnoFeatures>
  Material;
  Toolkit;
  Mvvm;
  ThemeService;
  SkiaRenderer;
</UnoFeatures>
```

**Enterprise App (Full-Featured):**
```xml
<UnoFeatures>
  Material;
  Hosting;
  Toolkit;
  Logging;
  MVUX;              <!-- Modern reactive pattern -->
  Configuration;
  HttpKiota;         <!-- API integration -->
  Serialization;
  Localization;
  Navigation;
  ThemeService;
  SkiaRenderer;
</UnoFeatures>
```

### Feature Selection Guidelines
- **Material** - Always include for Material Design 3 theme
- **Toolkit** - Required for AutoLayout, Card, NavigationBar, etc.
- **MVUX** - Modern reactive architecture (preferred for new apps)
- **Mvvm** - Traditional MVVM with CommunityToolkit.Mvvm
- **HttpKiota** - Preferred over Refit for new HTTP clients
- **SkiaRenderer** - Enables advanced rendering capabilities

### References
- Official docs: https://platform.uno/docs/articles/features/using-the-uno-sdk.html
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\HockeyBarn.csproj

## GlobalUsings Pattern

**When to apply:** Every project

### Standard Pattern
```csharp
// GlobalUsings.cs at project root
global using System.Collections.Immutable;
global using CommunityToolkit.Mvvm.ComponentModel;
global using CommunityToolkit.Mvvm.Input;
global using Microsoft.Extensions.DependencyInjection;
global using Microsoft.Extensions.Logging;
global using ApplicationExecutionState = Windows.ApplicationModel.Activation.ApplicationExecutionState;
```

### Principles
- Reduces per-file using statements
- Makes MVVM attributes available without prefix
- Include project-specific namespaces
- Never include platform-specific usings here

### References
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\GlobalUsings.cs
