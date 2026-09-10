# Build Configuration (Critical Priority)

## Directory.Build.props Hierarchy

**When to apply:** Multi-project solutions or shared build configuration

### Root Directory.Build.props Pattern
```xml
<Project>
  <PropertyGroup>
    <!-- Platform-specific build flags -->
    <Build_Android>$([MSBuild]::IsOSPlatform('windows'))</Build_Android>
    <Build_iOS>$([MSBuild]::IsOSPlatform('osx'))</Build_iOS>
    <Build_Windows>$([MSBuild]::IsOSPlatform('windows'))</Build_Windows>
    <Build_Desktop>true</Build_Desktop>
    <Build_Web>true</Build_Web>

    <!-- XAML validation -->
    <ShouldWriteErrorOnInvalidXaml>True</ShouldWriteErrorOnInvalidXaml>
  </PropertyGroup>

  <!-- Allow local overrides -->
  <Import Project="DebugPlatforms.props" Condition="Exists('DebugPlatforms.props')" />
</Project>
```

### Conditional TFM Assignment Pattern
```xml
<!-- In tfms-ui-winui.props or similar -->
<PropertyGroup>
  <TargetFrameworks></TargetFrameworks>
  <TargetFrameworks Condition="'$(Build_iOS)'=='true'">$(TargetFrameworks);net9.0-ios</TargetFrameworks>
  <TargetFrameworks Condition="'$(Build_Android)'=='true'">$(TargetFrameworks);net9.0-android</TargetFrameworks>
  <TargetFrameworks Condition="'$(Build_Windows)'=='true'">$(TargetFrameworks);net9.0-windows10.0.19041</TargetFrameworks>
  <TargetFrameworks Condition="'$(Build_Desktop)'=='true'">$(TargetFrameworks);net9.0-desktop</TargetFrameworks>
  <TargetFrameworks Condition="'$(Build_Web)'=='true'">$(TargetFrameworks);net9.0-browserwasm</TargetFrameworks>
  <TargetFrameworks>$(TargetFrameworks);net9.0</TargetFrameworks>
</PropertyGroup>
```

### Key Principles
- Net9.0 (or net8.0) is ALWAYS included as base TFM
- Other TFMs are conditionally added based on platform flags
- Developers can disable specific platforms via DebugPlatforms.props

### References
- Source: C:\Users\Platform006\uno.extensions\Directory.Build.props
- Source: C:\Users\Platform006\uno.extensions\src\tfms-ui-winui.props

## Central Package Management

**When to apply:** Multi-project solutions

### Directory.Packages.props Pattern
```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
    <CentralPackageTransitivePinningEnabled>true</CentralPackageTransitivePinningEnabled>
  </PropertyGroup>

  <ItemGroup>
    <!-- Core Uno Platform -->
    <PackageVersion Include="Uno.WinUI" Version="$(UnoVersion)" />

    <!-- Microsoft Extensions -->
    <PackageVersion Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
    <PackageVersion Include="Microsoft.Extensions.Logging" Version="8.0.0" />
    <PackageVersion Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />

    <!-- HTTP Clients -->
    <PackageVersion Include="Microsoft.Kiota.Abstractions" Version="1.16.4" />
    <PackageVersion Include="Refit" Version="7.2.22" />

    <!-- MVVM -->
    <PackageVersion Include="CommunityToolkit.Mvvm" Version="7.0.1" />
  </ItemGroup>
</Project>
```

### In Project Files
```xml
<ItemGroup>
  <!-- Version managed centrally -->
  <PackageReference Include="Uno.WinUI" />
  <PackageReference Include="Microsoft.Extensions.Hosting" />
</ItemGroup>
```

### Benefits
- Single source of truth for versions
- Prevents version conflicts across projects
- Transitive pinning prevents hidden conflicts

### References
- Source: C:\Users\Platform006\uno.extensions\Directory.Packages.props
- Source: C:\Users\Platform006\uno.extensions\src\Directory.Build.props

## Build Output Customization

**When to apply:** Multi-target projects to avoid output conflicts

### Pattern
```xml
<PropertyGroup>
  <BaseOutputPath>bin\$(MSBuildProjectName)</BaseOutputPath>
  <BaseIntermediateOutputPath>obj\$(MSBuildProjectName)</BaseIntermediateOutputPath>
</PropertyGroup>
```

### Benefits
- Each project outputs to its own folder
- Prevents bin/obj conflicts in multi-target scenarios
- Cleaner build output

### References
- Source: C:\Users\Platform006\uno.extensions\src\Directory.Build.props

## Build Performance Optimizations

**When to apply:** All projects

### Hard Link Strategy
```xml
<PropertyGroup>
  <CreateHardLinksForCopyFilesToOutputDirectoryIfPossible>true</CreateHardLinksForCopyFilesToOutputDirectoryIfPossible>
  <CreateHardLinksForCopyLocalIfPossible>true</CreateHardLinksForCopyLocalIfPossible>
  <CreateHardLinksForAdditionalFilesIfPossible>true</CreateHardLinksForAdditionalFilesIfPossible>
  <CreateHardLinksForPublishFilesIfPossible>true</CreateHardLinksForPublishFilesIfPossible>
</PropertyGroup>
```

### Binding Redirects
```xml
<PropertyGroup>
  <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
  <GenerateBindingRedirectsOutputType>true</GenerateBindingRedirectsOutputType>
</PropertyGroup>
```

### Benefits
- Reduces build times via hard links instead of file copying
- Automatically resolves assembly version conflicts

### References
- Source: C:\Users\Platform006\uno.extensions\src\Directory.Build.props

## Platform Detection Pattern

**When to apply:** Conditional compilation in .csproj files

### Pattern
```xml
<PropertyGroup>
  <_IsAndroid Condition="$(IsMonoAndroid) or $([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">true</_IsAndroid>
  <_IsiOS Condition="$(_IsiOS) or $([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">true</_IsiOS>
  <_IsCatalyst Condition="$(_IsCatalyst) or $([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'maccatalyst'">true</_IsCatalyst>
  <_IsWindows Condition="$(_IsWindows) or $([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">true</_IsWindows>
</PropertyGroup>
```

### Usage
```xml
<ItemGroup Condition="'$(_IsAndroid)' == 'true'">
  <PackageReference Include="Xamarin.AndroidX.Core" />
</ItemGroup>
```

### Principles
- Supports both legacy Xamarin (IsMonoAndroid) and new .NET 9 TFMs
- Use MSBuild functions for forward compatibility
- Define platform flags once, reuse throughout

### References
- Source: C:\Users\Platform006\uno.extensions\src\Directory.Build.props
