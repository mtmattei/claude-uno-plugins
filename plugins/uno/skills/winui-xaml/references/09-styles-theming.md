# Styles & Theming Skills - WinUI 3 XAML

## STYLES-001: Use Theme Resources for Colors

**Rule**: Never hardcode colors. Use `{ThemeResource}` for colors that should adapt to light/dark/high-contrast themes.

**Why**: Hardcoded colors break in different themes. ThemeResources automatically adapt, ensuring readability and accessibility across all theme modes.

**Example (Correct)**:
```xml
<Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
    <TextBlock Text="Title"
               Foreground="{ThemeResource TextFillColorPrimaryBrush}"/>
    <TextBlock Text="Subtitle"
               Foreground="{ThemeResource TextFillColorSecondaryBrush}"/>
    <Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
            BorderBrush="{ThemeResource CardStrokeColorDefaultBrush}">
        <!-- Card content -->
    </Border>
</Grid>
```

**Common Theme Resources**:
| Resource | Purpose |
|----------|---------|
| `ApplicationPageBackgroundThemeBrush` | Page background |
| `TextFillColorPrimaryBrush` | Primary text |
| `TextFillColorSecondaryBrush` | Secondary text |
| `TextFillColorTertiaryBrush` | Hint/placeholder text |
| `AccentFillColorDefaultBrush` | Accent/brand color |
| `SystemFillColorCriticalBrush` | Error states |
| `SystemFillColorSuccessBrush` | Success states |

**Common Mistakes**:
```xml
<!-- WRONG: Hardcoded colors -->
<Grid Background="#FFFFFF">
    <TextBlock Foreground="#000000"/>
</Grid>
<!-- Unreadable in dark theme! -->
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-theme-resources

---

## STYLES-002: Define Styles in ResourceDictionary

**Rule**: Define reusable styles in ResourceDictionary, either in App.xaml or separate .xaml files merged into App.xaml.

**Why**: Centralized styles ensure consistency, reduce duplication, and make global changes easy. Inline styles should be rare exceptions.

**Example (Correct)**:
```xml
<!-- App.xaml -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="Styles/Buttons.xaml"/>
            <ResourceDictionary Source="Styles/TextStyles.xaml"/>
        </ResourceDictionary.MergedDictionaries>

        <!-- App-wide styles -->
        <Style x:Key="PageHeaderStyle" TargetType="TextBlock">
            <Setter Property="FontSize" Value="28"/>
            <Setter Property="FontWeight" Value="SemiBold"/>
            <Setter Property="Foreground" Value="{ThemeResource TextFillColorPrimaryBrush}"/>
        </Style>
    </ResourceDictionary>
</Application.Resources>
```

```xml
<!-- Styles/Buttons.xaml -->
<ResourceDictionary>
    <Style x:Key="PrimaryButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="{ThemeResource AccentFillColorDefaultBrush}"/>
        <Setter Property="Foreground" Value="{ThemeResource TextOnAccentFillColorPrimaryBrush}"/>
        <Setter Property="CornerRadius" Value="4"/>
        <Setter Property="Padding" Value="16,8"/>
    </Style>
</ResourceDictionary>
```

**Usage**:
```xml
<TextBlock Text="Page Title" Style="{StaticResource PageHeaderStyle}"/>
<Button Content="Submit" Style="{StaticResource PrimaryButtonStyle}"/>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-resource-dictionaries

---

## STYLES-003: Use Lightweight Styling for Control Customization

**Rule**: Override theme resources at app/page level instead of creating complete custom templates.

**Why**: Lightweight styling preserves control behavior and accessibility while customizing appearance. Full control templates are fragile and require updates with WinUI versions.

**Example (Correct)**:
```xml
<!-- Override Button resources at app level -->
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Default">
                <CornerRadius x:Key="ControlCornerRadius">8</CornerRadius>
                <SolidColorBrush x:Key="ButtonBackground"
                                 Color="{ThemeResource AccentFillColorDefault}"/>
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <CornerRadius x:Key="ControlCornerRadius">8</CornerRadius>
                <SolidColorBrush x:Key="ButtonBackground"
                                 Color="{ThemeResource AccentFillColorDefault}"/>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

**Common Mistakes**:
```xml
<!-- WRONG: Full ControlTemplate is fragile and hard to maintain -->
<Style TargetType="Button">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <!-- 200 lines of template that will break with WinUI updates -->
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

**Uno Platform Notes**: Uno Themes (Material, Cupertino) use lightweight styling extensively. Override resources in ColorPaletteOverride.xaml.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-theme-resources#modify-the-resources-for-a-control

---

## STYLES-004: Support Light and Dark Themes

**Rule**: Define theme-specific resources in `ThemeDictionaries` with `Default`, `Dark`, and optionally `HighContrast` keys.

**Why**: Users expect apps to follow system theme. ThemeDictionaries provide automatic switching without code.

**Example (Correct)**:
```xml
<ResourceDictionary>
    <ResourceDictionary.ThemeDictionaries>
        <!-- Light theme (also used as fallback) -->
        <ResourceDictionary x:Key="Default">
            <SolidColorBrush x:Key="MyAppAccentBrush" Color="#0078D4"/>
            <SolidColorBrush x:Key="MyAppBackgroundBrush" Color="#F3F3F3"/>
        </ResourceDictionary>

        <!-- Dark theme -->
        <ResourceDictionary x:Key="Dark">
            <SolidColorBrush x:Key="MyAppAccentBrush" Color="#60CDFF"/>
            <SolidColorBrush x:Key="MyAppBackgroundBrush" Color="#202020"/>
        </ResourceDictionary>

        <!-- High Contrast -->
        <ResourceDictionary x:Key="HighContrast">
            <SolidColorBrush x:Key="MyAppAccentBrush"
                             Color="{ThemeResource SystemColorHighlightColor}"/>
            <SolidColorBrush x:Key="MyAppBackgroundBrush"
                             Color="{ThemeResource SystemColorWindowColor}"/>
        </ResourceDictionary>
    </ResourceDictionary.ThemeDictionaries>
</ResourceDictionary>
```

**Usage**:
```xml
<Grid Background="{ThemeResource MyAppBackgroundBrush}">
    <Button Background="{ThemeResource MyAppAccentBrush}"/>
</Grid>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-theme-resources#theme-specific-dictionaries

---

## STYLES-005: Use BasedOn for Style Inheritance

**Rule**: Use `BasedOn` to extend existing styles rather than duplicating all setters.

**Why**: Style inheritance reduces duplication, ensures consistency with base style updates, and makes maintenance easier.

**Example (Correct)**:
```xml
<!-- Base button style -->
<Style x:Key="BaseButtonStyle" TargetType="Button">
    <Setter Property="CornerRadius" Value="4"/>
    <Setter Property="Padding" Value="16,8"/>
    <Setter Property="MinHeight" Value="32"/>
</Style>

<!-- Primary button extends base -->
<Style x:Key="PrimaryButtonStyle" TargetType="Button"
       BasedOn="{StaticResource BaseButtonStyle}">
    <Setter Property="Background" Value="{ThemeResource AccentFillColorDefaultBrush}"/>
    <Setter Property="Foreground" Value="{ThemeResource TextOnAccentFillColorPrimaryBrush}"/>
</Style>

<!-- Secondary button extends base -->
<Style x:Key="SecondaryButtonStyle" TargetType="Button"
       BasedOn="{StaticResource BaseButtonStyle}">
    <Setter Property="Background" Value="Transparent"/>
    <Setter Property="BorderBrush" Value="{ThemeResource ControlStrokeColorDefaultBrush}"/>
    <Setter Property="BorderThickness" Value="1"/>
</Style>
```

**Common Mistakes**:
```xml
<!-- WRONG: Duplicated properties in both styles -->
<Style x:Key="PrimaryButtonStyle" TargetType="Button">
    <Setter Property="CornerRadius" Value="4"/>
    <Setter Property="Padding" Value="16,8"/>
    <Setter Property="MinHeight" Value="32"/>
    <Setter Property="Background" Value="{ThemeResource AccentFillColorDefaultBrush}"/>
</Style>

<Style x:Key="SecondaryButtonStyle" TargetType="Button">
    <Setter Property="CornerRadius" Value="4"/>  <!-- Duplicated -->
    <Setter Property="Padding" Value="16,8"/>    <!-- Duplicated -->
    <Setter Property="MinHeight" Value="32"/>    <!-- Duplicated -->
    <Setter Property="Background" Value="Transparent"/>
</Style>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-styles#apply-an-implicit-style

---

## STYLES-006: Use Implicit Styles Sparingly

**Rule**: Use implicit styles (no x:Key) only for truly global defaults. Prefer explicit keyed styles for flexibility.

**Why**: Implicit styles apply to ALL controls of that type, which can cause unexpected styling. Keyed styles are explicit and intentional.

**Example (Correct)**:
```xml
<!-- Explicit style - applied only where specified -->
<Style x:Key="CardTextStyle" TargetType="TextBlock">
    <Setter Property="FontSize" Value="14"/>
    <Setter Property="TextWrapping" Value="Wrap"/>
</Style>

<!-- Implicit style - use only for true app-wide defaults -->
<Style TargetType="Page">
    <Setter Property="Background" Value="{ThemeResource ApplicationPageBackgroundThemeBrush}"/>
</Style>
```

**When implicit styles are appropriate**:
- Page background color
- Default font family (app-wide)
- Scroll behavior on ScrollViewer

**Common Mistakes**:
```xml
<!-- WRONG: Implicit style affects ALL TextBlocks unexpectedly -->
<Style TargetType="TextBlock">
    <Setter Property="FontSize" Value="16"/>
    <Setter Property="Foreground" Value="Blue"/>
</Style>
<!-- Now every TextBlock, including in controls like Button, is blue -->
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-styles#apply-an-implicit-style

---

## STYLES-007: Use StaticResource vs ThemeResource Correctly

**Rule**: Use `{ThemeResource}` for theme-aware resources. Use `{StaticResource}` for resources that don't change with theme.

**Why**: ThemeResource re-evaluates when theme changes. StaticResource is resolved once at load time. Using StaticResource for theme colors breaks theme switching.

**Example (Correct)**:
```xml
<!-- ThemeResource for theme-aware colors -->
<TextBlock Foreground="{ThemeResource TextFillColorPrimaryBrush}"/>
<Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"/>

<!-- StaticResource for non-theme resources -->
<TextBlock Style="{StaticResource TitleTextBlockStyle}"/>
<Image Source="{StaticResource AppLogoImage}"/>
<Grid Margin="{StaticResource PagePadding}"/>
```

**Common Mistakes**:
```xml
<!-- WRONG: StaticResource for theme color won't update on theme change -->
<TextBlock Foreground="{StaticResource TextFillColorPrimaryBrush}"/>

<!-- WRONG: ThemeResource for non-theme resource - unnecessary overhead -->
<TextBlock Style="{ThemeResource TitleTextBlockStyle}"/>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/style/xaml-theme-resources#themeresource-vs-staticresource
