# Navigation Patterns (High Priority)

## Navigation Setup with Uno.Extensions.Navigation

**When to apply:** Multi-page applications

### App.xaml.cs Setup
```csharp
protected override async void OnLaunched(LaunchActivatedEventArgs args)
{
    var builder = this.CreateBuilder(args)
        .Configure(host => host
            .UseNavigation(RegisterRoutes)
        );

    MainWindow = builder.Window;
    Host = await builder.NavigateAsync<Shell>();
}

private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        new ViewMap(ViewModel: typeof(ShellViewModel)),
        new ViewMap<MainPage, MainViewModel>(),
        new ViewMap<DetailPage, DetailViewModel>(),
        new DataViewMap<ItemDetailPage, ItemDetailViewModel, Item>()
    );

    routes.Register(
        new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
            Nested:
            [
                new ("Main", View: views.FindByView<MainPage>(), IsDefault: true),
                new ("Detail", View: views.FindByView<DetailPage>()),
                new ("ItemDetail", View: views.FindByView<ItemDetailPage>())
            ]
        )
    );
}
```

### ViewMap Types
- `ViewMap(ViewModel: typeof(VM))` - View-ViewModel association without type params
- `ViewMap<View, ViewModel>()` - Typed View-ViewModel association
- `DataViewMap<View, ViewModel, Data>()` - With navigation data passing

### RouteMap Structure
- **""** - Root route (Shell)
- **Nested** - Child routes within Shell
- **IsDefault: true** - Default child route
- **View** - Reference to registered ViewMap

### References
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Walkthrough/RegisterRoutes.html
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\App.xaml.cs

## Shell Pattern with Frame

**When to apply:** All multi-page apps

### Shell.xaml
```xml
<UserControl x:Class="MyApp.Shell"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:utu="using:Uno.Toolkit.UI">

    <utu:ExtendedSplashScreen x:Name="Splash"
                              HorizontalAlignment="Stretch"
                              VerticalAlignment="Stretch">
        <utu:ExtendedSplashScreen.LoadingContent>
            <Grid Background="{ThemeResource BackgroundColor}">
                <!-- Splash screen content -->
            </Grid>
        </utu:ExtendedSplashScreen.LoadingContent>

        <utu:ExtendedSplashScreen.Content>
            <Frame x:Name="ShellFrame" />
        </utu:ExtendedSplashScreen.Content>
    </utu:ExtendedSplashScreen>

</UserControl>
```

### Shell.xaml.cs
```csharp
public sealed partial class Shell : UserControl
{
    public Shell()
    {
        this.InitializeComponent();
    }

    public Frame RootFrame => ShellFrame;
}
```

### References
- Example: C:\Users\Platform006\source\repos\UnoApp3\UnoApp3\Shell.xaml

## Navigation via INavigator (Code-Behind)

**When to apply:** When XAML-based navigation is not sufficient

### ViewModel Navigation
```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly INavigator _navigator;

    public MainViewModel(INavigator navigator)
    {
        _navigator = navigator;
    }

    [RelayCommand]
    private async Task NavigateToDetail()
    {
        await _navigator.NavigateViewModelAsync<DetailViewModel>(this);
    }

    [RelayCommand]
    private async Task NavigateWithData(Item item)
    {
        await _navigator.NavigateViewModelAsync<ItemDetailViewModel>(
            this,
            data: item);
    }

    [RelayCommand]
    private async Task GoBack()
    {
        await _navigator.NavigateBackAsync(this);
    }
}
```

### Navigation Methods
- `NavigateViewModelAsync<TViewModel>(sender)` - Navigate to ViewModel
- `NavigateViewModelAsync<TViewModel>(sender, data: obj)` - Navigate with data
- `NavigateBackAsync(sender)` - Go back in stack
- `NavigateRouteAsync(sender, "RouteName")` - Navigate by route name

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\Presentation\MainViewModel.cs
- Official docs: https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/NavigationCodeBehind.html

## XAML-Based Navigation (Preferred)

**When to apply:** Whenever possible (more declarative)

### Navigation.Request Pattern
```xml
<Button Content="Go to Detail"
        utu:Navigation.Request="Detail" />

<Button Content="Go Back"
        utu:Navigation.Request="-" />

<Button Content="Go to Settings"
        utu:Navigation.Request="./Settings" />
```

### Navigation Data Passing
```xml
<ListView ItemsSource="{x:Bind ViewModel.Items}">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="models:Item">
            <Border utu:Navigation.Request="ItemDetail"
                    utu:Navigation.Data="{x:Bind}">
                <TextBlock Text="{x:Bind Name}" />
            </Border>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

### Navigation Syntax
- `"Detail"` - Navigate to Detail route
- `"-"` - Go back
- `"./Settings"` - Navigate to sibling route
- `"../Profile"` - Navigate to parent then Profile

### References
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/HowTo-NavigateBetweenPages.html

## Receiving Navigation Data

**When to apply:** When page receives data from navigation

### ViewModel Pattern
```csharp
public partial class ItemDetailViewModel : ObservableObject
{
    private readonly INavigator _navigator;
    private readonly Item _item;

    public ItemDetailViewModel(
        INavigator navigator,
        Item item)  // Injected via DataViewMap
    {
        _navigator = navigator;
        _item = item;

        Title = item.Name;
        Description = item.Description;
    }

    [ObservableProperty]
    private string? _title;

    [ObservableProperty]
    private string? _description;
}
```

### Registration for Data
```csharp
views.Register(
    new DataViewMap<ItemDetailPage, ItemDetailViewModel, Item>()
);
```

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\Presentation
- Official docs: https://platform.uno/docs/articles/external/uno.chefs/doc/navigation/NavigationCodeBehind.html

## TabBar Navigation Pattern

**When to apply:** Bottom navigation or tab-based navigation

### Shell with TabBar
```xml
<utu:TabBar x:Name="MainTabBar"
            HorizontalAlignment="Stretch"
            VerticalAlignment="Bottom">

    <utu:TabBar.Items>
        <utu:TabBarItem Content="Home"
                        Icon="Home"
                        utu:Navigation.Request="Home" />

        <utu:TabBarItem Content="Search"
                        Icon="Find"
                        utu:Navigation.Request="Search" />

        <utu:TabBarItem Content="Profile"
                        Icon="Contact"
                        utu:Navigation.Request="Profile" />
    </utu:TabBar.Items>

</utu:TabBar>
```

### Route Configuration
```csharp
routes.Register(
    new RouteMap("", View: views.FindByViewModel<ShellViewModel>(),
        Nested:
        [
            new ("Home", View: views.FindByView<HomePage>(), IsDefault: true),
            new ("Search", View: views.FindByView<SearchPage>()),
            new ("Profile", View: views.FindByView<ProfilePage>())
        ]
    )
);
```

### References
- Uno Toolkit docs: TabBar control
- Navigation integration pattern from HockeyBarn app structure
