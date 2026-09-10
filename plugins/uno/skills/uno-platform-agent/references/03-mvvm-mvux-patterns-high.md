# MVVM & MVUX Patterns (High Priority)

## MVVM with CommunityToolkit.Mvvm

**When to apply:** Traditional MVVM pattern with mutable state

### ViewModel Pattern
```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly INavigator _navigator;

    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(DisplayName))]
    private string? _name;

    public string DisplayName => $"Hello, {Name}!";

    [ObservableProperty]
    private bool _isLoading;

    public MainViewModel(INavigator navigator, IStringLocalizer localizer)
    {
        _navigator = navigator;
        SaveCommand = new AsyncRelayCommand(SaveAsync);
    }

    public ICommand SaveCommand { get; }

    private async Task SaveAsync()
    {
        IsLoading = true;
        try
        {
            await _service.SaveAsync(Name);
        }
        finally
        {
            IsLoading = false;
        }
    }

    [RelayCommand]
    private async Task NavigateToDetail()
    {
        await _navigator.NavigateViewModelAsync<DetailViewModel>(this);
    }

    partial void OnNameChanged(string? value)
    {
        // Side effects when Name changes
        OnPropertyChanged(nameof(DisplayName));
    }
}
```

### Key Patterns
- `[ObservableProperty]` generates full property with INotifyPropertyChanged
- `[NotifyPropertyChangedFor]` automatically notifies dependent properties
- `[RelayCommand]` generates ICommand from methods
- `partial void OnPropertyChanged(Type value)` for side effects
- Dependency injection in constructor

### XAML Binding
```xml
<TextBox Text="{x:Bind ViewModel.Name, Mode=TwoWay}" />
<TextBlock Text="{x:Bind ViewModel.DisplayName, Mode=OneWay}" />
<Button Content="Save" Command="{x:Bind ViewModel.SaveCommand}" />
<Button Content="Navigate" Command="{x:Bind ViewModel.NavigateToDetailCommand}" />
```

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\Presentation\MainViewModel.cs
- Example: C:\Users\Platform006\source\repos\FibonacciSphere\FibonacciSphere\ViewModels\SphereViewModel.cs

## MVUX Reactive Pattern

**When to apply:** Modern reactive architecture with immutable state (Preferred for new apps)

### Model Pattern (MVUX)
```csharp
public partial record RecipesModel(IRecipeService RecipeService)
{
    public IListFeed<Recipe> Recipes => ListFeed.Async(RecipeService.GetRecipes);

    public async ValueTask<Recipe> GetRecipeDetails(Recipe recipe, CancellationToken ct)
    {
        return await RecipeService.GetRecipeDetails(recipe.Id, ct);
    }

    public async ValueTask SaveRecipe(Recipe recipe, CancellationToken ct)
    {
        await RecipeService.SaveAsync(recipe, ct);
    }
}
```

### Generated ViewModel (Automatic)
MVUX generates:
- `IListFeed<Recipe> Recipes` -> Observable property with loading/error states
- `GetRecipeDetailsCommand` -> IAsyncCommand from GetRecipeDetails method
- `SaveRecipeCommand` -> IAsyncCommand from SaveRecipe method

### XAML Binding
```xml
<ListView ItemsSource="{x:Bind ViewModel.Recipes}" />
<Button Command="{x:Bind ViewModel.GetRecipeDetailsCommand}"
        CommandParameter="{x:Bind SelectedRecipe}" />
<Button Command="{x:Bind ViewModel.SaveRecipeCommand}"
        CommandParameter="{x:Bind CurrentRecipe}" />
```

### Feed Types

**IListFeed<T>** - Read-only observable collection
```csharp
public IListFeed<Product> Products => ListFeed.Async(ProductService.GetProducts);
```

**IListState<T>** - Mutable observable collection
```csharp
public IListState<Product> Products => ListState.Async(this, ProductService.GetProducts);
```

**IFeed<T>** - Read-only single value
```csharp
public IFeed<User> CurrentUser => Feed.Async(UserService.GetCurrentUser);
```

**IState<T>** - Mutable single value
```csharp
public IState<string> SearchQuery => State.Value(this, () => string.Empty);
```

### When to Use Feeds vs States
- Use **IListFeed** when data is read-only and pulled from service
- Use **IListState** when you need to edit the collection
- Use **IFeed** for computed/derived read-only values
- Use **IState** for mutable values that change over time

### Disabling Command Generation
```csharp
[ImplicitCommand(false)]
public async ValueTask MyMethod() { }
```

### References
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Reference/Reactive/Architecture.html
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Walkthrough/ListFeed.howto.html
- Example: C:\Users\Platform006\source\repos\XamlBindingBlogSample (uses MVUX)

## Data Binding Conventions

**When to apply:** All XAML bindings

### x:Bind vs Binding
```xml
<!-- Prefer x:Bind (compile-time, better performance) -->
<TextBlock Text="{x:Bind ViewModel.Name, Mode=OneWay}" />

<!-- Only use {Binding} for legacy scenarios -->
<TextBlock Text="{Binding Name}" />
```

### Binding Modes
- **Mode=OneWay** - Display-only (default for x:Bind properties)
- **Mode=TwoWay** - User input controls (TextBox, Slider, ComboBox)
- **Mode=OneTime** - Set once at load

### String Formatting
```xml
<!-- WRONG: StringFormat not supported in Uno Platform -->
<TextBlock Text="{x:Bind ViewModel.Price, Mode=OneWay, StringFormat='${0:F2}'}" />

<!-- CORRECT: Use multiple Run elements -->
<TextBlock>
    <Run Text="$" />
    <Run Text="{x:Bind ViewModel.FormattedPrice, Mode=OneWay}" />
</TextBlock>
```

Or create computed property in ViewModel:
```csharp
public string FormattedPrice => $"${Price:F2}";
```

### Visibility Binding
```xml
<!-- Bool to Visibility is implicit -->
<TextBlock Text="Loading..." Visibility="{x:Bind ViewModel.IsLoading, Mode=OneWay}" />
```

### References
- Example: C:\Users\Platform006\source\repos\FibonacciSphere\FibonacciSphere\MainPage.xaml
- Official docs: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Walkthrough/Commands.howto.html

## Dependency Injection Pattern

**When to apply:** All services and ViewModels

### Service Registration in App.xaml.cs
```csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    var builder = this.CreateBuilder(args)
        .Configure(host => host
#if DEBUG
            .UseEnvironment(Environments.Development)
#endif
            .UseLogging(configure: (context, logBuilder) => {
                logBuilder.SetMinimumLevel(LogLevel.Warning);
#if DEBUG
                logBuilder.SetMinimumLevel(LogLevel.Debug);
#endif
            })
            .ConfigureServices((context, services) => {
                // Register services
                services.AddSingleton<IDatabaseService, DatabaseService>();
                services.AddTransient<IApiService, ApiService>();

                // Register ViewModels
                services.AddTransient<MainViewModel>();
                services.AddTransient<DetailViewModel>();
            })
            .UseNavigation(RegisterRoutes)
        );

    MainWindow = builder.Window;
    Host = await builder.NavigateAsync<Shell>();
}
```

### ViewModel with DI
```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly INavigator _navigator;
    private readonly IApiService _apiService;
    private readonly IStringLocalizer _localizer;

    public MainViewModel(
        INavigator navigator,
        IApiService apiService,
        IStringLocalizer localizer,
        IOptions<AppConfig> config)
    {
        _navigator = navigator;
        _apiService = apiService;
        _localizer = localizer;

        Title = $"{_localizer["Welcome"]} - {config.Value.Environment}";
    }
}
```

### References
- Example: C:\Users\Platform006\source\repos\HockeyBarn\HockeyBarn\App.xaml.cs
- Source: C:\Users\Platform006\uno.extensions\src\Uno.Extensions.Hosting\HostBuilderExtensions.cs
