# Styling & Theming (Medium Priority)

## Material Theme Setup

**When to apply:** All apps using Material Design

### App.xaml Pattern
```xml
<Application x:Class="MyApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:utum="using:Uno.Toolkit.UI.Material">

    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <XamlControlsResources xmlns="using:Microsoft.UI.Xaml.Controls" />

                <utum:MaterialToolkitTheme
                    ColorOverrideSource="ms-appx:///Styles/ColorPaletteOverride.xaml">
                </utum:MaterialToolkitTheme>

            </ResourceDictionary.MergedDictionaries>

            <!-- App-level resources here -->

        </ResourceDictionary>
    </Application.Resources>

</Application>
```

### UnoFeatures Requirement
```xml
<UnoFeatures>
  Material;
  Toolkit;
  <!-- other features -->
</UnoFeatures>
```

### References
- Official docs: https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/material-getting-started.html
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\App.xaml

## Color Palette Override

**When to apply:** Custom brand colors

### Styles/ColorPaletteOverride.xaml Pattern
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">

    <!-- Primary Colors -->
    <Color x:Key="PrimaryColor">#FF6B35</Color>
    <Color x:Key="OnPrimaryColor">#FFFFFF</Color>
    <Color x:Key="PrimaryContainerColor">#FFD7CC</Color>
    <Color x:Key="OnPrimaryContainerColor">#2C1410</Color>

    <!-- Secondary Colors -->
    <Color x:Key="SecondaryColor">#00A8E8</Color>
    <Color x:Key="OnSecondaryColor">#FFFFFF</Color>
    <Color x:Key="SecondaryContainerColor">#C7E7F7</Color>
    <Color x:Key="OnSecondaryContainerColor">#001F2A</Color>

    <!-- Tertiary Colors -->
    <Color x:Key="TertiaryColor">#7B68EE</Color>

    <!-- Surface Colors -->
    <Color x:Key="SurfaceColor">#FFFBFF</Color>
    <Color x:Key="OnSurfaceColor">#1C1B1E</Color>
    <Color x:Key="SurfaceVariantColor">#E7E0EB</Color>
    <Color x:Key="OnSurfaceVariantColor">#49454E</Color>

    <!-- Background Colors -->
    <Color x:Key="BackgroundColor">#FFFBFF</Color>
    <Color x:Key="OnBackgroundColor">#1C1B1E</Color>

    <!-- Error Colors -->
    <Color x:Key="ErrorColor">#BA1A1A</Color>
    <Color x:Key="OnErrorColor">#FFFFFF</Color>

</ResourceDictionary>
```

### Naming Convention
- **[Semantic]Color** - Base color (e.g., PrimaryColor, SurfaceColor)
- **On[Semantic]Color** - Text/icon color on the semantic color
- **[Semantic]ContainerColor** - Container/background variant
- **On[Semantic]ContainerColor** - Text/icon on container

### Never Hardcode Colors
```xml
<!-- WRONG -->
<Border Background="#FF6B35" />

<!-- CORRECT — use {Role}Brush for Brush properties, {Role}Color for Color properties -->
<Border Background="{ThemeResource PrimaryBrush}" />
```

### References
- Material Design 3 Theme Builder: https://m3.material.io/theme-builder
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\Styles\ColorPaletteOverride.xaml

## App-Level Style Resources

**When to apply:** Reusable styles across app

### App.xaml Resources Pattern
```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <XamlControlsResources />
            <utum:MaterialToolkitTheme ColorOverrideSource="..." />
        </ResourceDictionary.MergedDictionaries>

        <!-- Custom Brushes -->
        <SolidColorBrush x:Key="IceBlueBrush" Color="{ThemeResource PrimaryColor}" />
        <SolidColorBrush x:Key="OrangeBrush" Color="{ThemeResource SecondaryColor}" />

        <!-- Converters -->
        <converters:SkillLevelColorConverter x:Key="SkillLevelColorConverter" />
        <converters:BoolToVisibilityConverter x:Key="BoolToVisibilityConverter" />

        <!-- Card Style -->
        <Style x:Key="GlassCardStyle" TargetType="Border">
            <Setter Property="Background" Value="{ThemeResource SurfaceColor}" />
            <Setter Property="CornerRadius" Value="24" />
            <Setter Property="Padding" Value="24" />
            <Setter Property="Translation" Value="0,0,12" />
            <Setter Property="Shadow">
                <Setter.Value>
                    <ThemeShadow />
                </Setter.Value>
            </Setter>
        </Style>

        <!-- Primary Button Style -->
        <Style x:Key="PrimaryButtonStyle" TargetType="Button"
               BasedOn="{StaticResource FilledButtonStyle}">
            <Setter Property="Background" Value="{ThemeResource PrimaryBrush}" />
            <Setter Property="Foreground" Value="{ThemeResource OnPrimaryBrush}" />
            <Setter Property="Height" Value="56" />
            <Setter Property="CornerRadius" Value="28" />
            <Setter Property="Translation" Value="0,0,4" />
            <Setter Property="Shadow">
                <Setter.Value>
                    <ThemeShadow />
                </Setter.Value>
            </Setter>
        </Style>

        <!-- Secondary Button Style -->
        <Style x:Key="SecondaryButtonStyle" TargetType="Button"
               BasedOn="{StaticResource OutlinedButtonStyle}">
            <Setter Property="BorderBrush" Value="{ThemeResource PrimaryBrush}" />
            <Setter Property="Foreground" Value="{ThemeResource PrimaryBrush}" />
            <Setter Property="Height" Value="56" />
            <Setter Property="CornerRadius" Value="28" />
        </Style>

    </ResourceDictionary>
</Application.Resources>
```

### Key Principles
- Register converters in App.xaml (not page resources)
- Extend Material styles with `BasedOn`
- Use ThemeShadow for elevation
- Reference ThemeResource colors

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\App.xaml

## Elevation with ThemeShadow

**When to apply:** Cards, elevated buttons, focal elements

### Pattern
```xml
<Border CornerRadius="16"
        Background="{ThemeResource SurfaceColor}"
        Translation="0,0,32">
    <Border.Shadow>
        <ThemeShadow />
    </Border.Shadow>

    <!-- Content -->
    <StackPanel Padding="16" Spacing="8">
        <TextBlock Text="Elevated Card" />
    </StackPanel>
</Border>
```

### Z-Values for Translation
- 4-8: Subtle elevation (buttons, chips)
- 12-16: Cards, containers
- 24-32: Important focal elements, hero images
- 48+: Modal overlays, dialogs

### Benefits
- Consistent across all platforms
- No SVG shadows needed
- Performance optimized

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\Presentation\WelcomePage.xaml

## Custom Value Converters

**When to apply:** Complex UI logic in bindings

### Converter Pattern
```csharp
public class SkillLevelColorConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, string language)
    {
        if (value is SkillLevel skillLevel)
        {
            return skillLevel switch
            {
                SkillLevel.Beginner => Application.Current.Resources["TertiaryContainerColor"],
                SkillLevel.Intermediate => Application.Current.Resources["PrimaryContainerColor"],
                SkillLevel.Advanced => Application.Current.Resources["SecondaryContainerColor"],
                _ => Application.Current.Resources["SurfaceVariantColor"]
            };
        }
        return Application.Current.Resources["SurfaceVariantColor"];
    }

    public object ConvertBack(object value, Type targetType, object parameter, string language)
    {
        throw new NotImplementedException();
    }
}
```

### Usage
```xml
<!-- Register in App.xaml -->
<Application.Resources>
    <converters:SkillLevelColorConverter x:Key="SkillLevelColorConverter" />
</Application.Resources>

<!-- Use in binding -->
<Border Background="{x:Bind ViewModel.SkillLevel, Converter={StaticResource SkillLevelColorConverter}}" />
```

### Principles
- Return resource references, not hardcoded values
- One-way converters throw NotImplementedException for ConvertBack
- Access Application.Current.Resources for semantic colors

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\Converters\ValueConverters.cs

## Typography Styles

**When to apply:** Text hierarchy

### Do Not Set Font Sizes Directly
```xml
<!-- WRONG: Hardcoded font size -->
<TextBlock Text="Title" FontSize="24" FontWeight="Bold" />

<!-- CORRECT: Use predefined style -->
<TextBlock Text="Title" Style="{StaticResource TitleLargeTextBlockStyle}" />
```

### Common Material Styles
- **DisplayLarge** - 57sp
- **DisplayMedium** - 45sp
- **DisplaySmall** - 36sp
- **HeadlineLarge** - 32sp
- **HeadlineMedium** - 28sp
- **HeadlineSmall** - 24sp
- **TitleLarge** - 22sp
- **TitleMedium** - 16sp
- **TitleSmall** - 14sp
- **BodyLarge** - 16sp
- **BodyMedium** - 14sp (default)
- **BodySmall** - 12sp
- **LabelLarge** - 14sp
- **LabelMedium** - 12sp
- **LabelSmall** - 11sp

### References
- Material Design 3 Type Scale: https://m3.material.io/styles/typography/overview
- Uno Material theme provides all styles

## Responsive Design with Responsive Extension

**When to apply:** Adaptive layouts for different screen sizes

### Pattern
```xml
<Grid ColumnSpacing="{utu:Responsive Desktop=24, Tablet=16, Phone=12}">

    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="{utu:Responsive Desktop='1*', Tablet='1*', Phone='0'}" />
        <ColumnDefinition Width="{utu:Responsive Desktop='2*', Tablet='1*', Phone='1*'}" />
    </Grid.ColumnDefinitions>

    <TextBlock Text="Title"
               FontSize="{utu:Responsive Desktop=32, Tablet=24, Phone=20}" />

</Grid>
```

### Breakpoints
- **Phone** - 0-599px
- **Tablet** - 600-904px
- **Desktop** - 905px+

### References
- Official docs: Uno Toolkit Responsive extension
- Example pattern from FibonacciSphere responsive controls

## AutoLayout for Consistent Spacing

**When to apply:** All layouts (preferred over StackPanel)

### Pattern
```xml
<utu:AutoLayout Orientation="Vertical"
                Padding="24"
                Spacing="16">

    <TextBlock Text="Header" Style="{StaticResource HeadlineSmall}" />

    <!-- Nested horizontal -->
    <utu:AutoLayout Orientation="Horizontal" Spacing="8">
        <Button Content="Cancel" />
        <Button Content="Save" Style="{StaticResource PrimaryButtonStyle}" />
    </utu:AutoLayout>

    <!-- More content -->

</utu:AutoLayout>
```

### Spacing Scale
- 4, 8, 12, 16, 24, 32, 48, 64 (multiples of 4/8)

### Benefits
- Consistent spacing without per-child margins
- Cleaner XAML
- Better performance than StackPanel

### References
- Uno Toolkit docs: AutoLayout control
- Example: C:\Users\Platform006\source\repos\FibonacciSphere\FibonacciSphere\MainPage.xaml
