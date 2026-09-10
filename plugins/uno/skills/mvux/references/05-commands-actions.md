# Commands and Messaging

How MVUX auto-generates commands from model methods, and how to sync states across views using messaging.

---

## Implicit Command Generation

**Impact**: HIGH — Any public method on a Model automatically becomes an IAsyncCommand on the ViewModel.

### Basic Commands

```csharp
public partial record MainModel(IMyService Service)
{
    // Any of these generate a 'DoWork' command:
    public void DoWork() { ... }
    public ValueTask DoWork(CancellationToken ct) { ... }
    public Task DoWork() { ... }
}
```

Generated ViewModel:

```csharp
public IAsyncCommand DoWork { get; }
```

XAML binding:

```xml
<Button Command="{Binding DoWork}" Content="Do Work" />
```

During execution, the Button is automatically disabled (prevents double-tap).

### Command Generation Rules

A method generates a command if:
1. It returns `void`, `ValueTask`, or `Task`
2. It may have one `CancellationToken` parameter
3. It may have parameters resolved from feeds (see below)
4. It may have one additional parameter for `CommandParameter`
5. Methods with return values (non-void/Task/ValueTask) do NOT generate commands

### CommandParameter

```csharp
public void DoWork(double param) { ... }
```

```xml
<Slider x:Name="slider" Minimum="1" Maximum="100" />
<Button Command="{Binding DoWork}"
        CommandParameter="{Binding Value, ElementName=slider}" />
```

If `CommandParameter` is null or wrong type → button stays disabled.

**Best practice**: make methods async with `CancellationToken`:

```csharp
public ValueTask DoWork(double param, CancellationToken ct) { ... }
```

### Feed Parameters (Auto-Resolution)

If a method parameter's **name and type** match a Feed property (case-insensitive), the current feed value is injected automatically:

```csharp
public IFeed<int> CounterValue => ...

// 'counterValue' matches CounterValue feed by name + type
public void ResetCounter(int counterValue) { ... }
```

```xml
<!-- No CommandParameter needed — value comes from the feed -->
<Button Command="{Binding ResetCounter}" Content="Reset" />
```

Explicit matching with `[FeedParameter]`:

```csharp
public IFeed<string> Message { get; }

// Explicit: 'msg' doesn't match by name, so we specify
[ImplicitFeedCommandParameter(false)]
public async ValueTask Share([FeedParameter(nameof(Message))] string msg) { ... }
```

### Controlling Command Generation

**Per-method:**

```csharp
// Disable command for this method (still available as a regular method on ViewModel)
[Command(false)]
public async ValueTask Save() { ... }

// Force command generation even if implicit is disabled
[Command]
public async ValueTask DoWork() { ... }
```

**Per-class:**

```csharp
[ImplicitCommands(false)]
public partial record MyModel(...)
{
    [Command]  // Opt back in for this one method
    public async ValueTask Save() { ... }
}
```

**Per-assembly:**

```csharp
[assembly: ImplicitCommands(false)]
```

### Using x:Bind with Methods

When using `x:Bind` event binding, disable command generation for that method:

```csharp
[Command(false)]
public async ValueTask Save() { ... }
```

```xml
<Button Click="{x:Bind Save}" Content="Save" />
```

### Explicit Command Creation

For manual control:

```csharp
public ICommand MyCommand => Command.Async(async ct => await PingServer(ct));
```

Advanced (fluent API):

```csharp
public IFeed<int> PageCount => ...

public IAsyncCommand MyCommand => Command.Create(builder =>
    builder
        .Given(PageCount)
        .When(count => count > 0)
        .Then(async (count, ct) => await LoadPage(count, ct)));
```

### XAML Behaviors for Event-Based Commands

```xml
<TextBlock x:Name="textBlock" Text="Double-tap me">
    <interactivity:Interaction.Behaviors>
        <interactions:EventTriggerBehavior EventName="DoubleTapped">
            <interactions:InvokeCommandAction
                Command="{Binding TextBlockDoubleTapped}"
                CommandParameter="{Binding Text, ElementName=textBlock}" />
        </interactions:EventTriggerBehavior>
    </interactivity:Interaction.Behaviors>
</TextBlock>
```

Requires NuGet: `Uno.Microsoft.Xaml.Behaviors.Interactivity.WinUI` and `Uno.Microsoft.Xaml.Behaviors.WinUI.Managed`.

---

## Messaging (CRUD Sync)

**Impact**: HIGH — Keeps multiple views in sync when entities are created/updated/deleted.

### Architecture

1. **Service** performs CRUD and sends `EntityMessage<T>` via `IMessenger`
2. **Model** uses `.Observe(messenger)` on a state to auto-apply changes
3. Changes flow: Service → Messenger → Model's State → View (automatic)

### Setup

Register `IMessenger` as a singleton in DI:

```csharp
// In App.xaml.cs or host builder
services.AddSingleton<IMessenger, WeakReferenceMessenger>();
```

Namespace: `Uno.Extensions.Reactive.Messaging`

### Service — Sending Entity Messages

```csharp
using CommunityToolkit.Mvvm.Messaging;

public class PeopleService : IPeopleService
{
    private readonly IMessenger _messenger;

    public PeopleService(IMessenger messenger) => _messenger = messenger;

    public async ValueTask CreateAsync(Person person, CancellationToken ct)
    {
        var created = await _repository.CreateAsync(person, ct);
        _messenger.Send(new EntityMessage<Person>(EntityChange.Created, created));
    }

    public async ValueTask UpdateAsync(Person person, CancellationToken ct)
    {
        var updated = await _repository.UpdateAsync(person, ct);
        _messenger.Send(new EntityMessage<Person>(EntityChange.Updated, updated));
    }

    public async ValueTask DeleteAsync(Person person, CancellationToken ct)
    {
        await _repository.DeleteAsync(person, ct);
        _messenger.Send(new EntityMessage<Person>(EntityChange.Deleted, person));
    }
}
```

### Model — Observing Entity Messages

```csharp
using CommunityToolkit.Mvvm.Messaging;
using Uno.Extensions.Reactive.Messaging;

public partial record PeopleModel
{
    protected IPeopleService PeopleService { get; }

    public IListState<Person> People =>
        ListState.Async(this, PeopleService.GetAllAsync);

    public PeopleModel(IPeopleService peopleService, IMessenger messenger)
    {
        PeopleService = peopleService;

        // Auto-apply Created/Updated/Deleted messages to the People list
        messenger.Observe(state: People, keySelector: person => person.Id);
    }
}
```

The `keySelector` tells Observe how to match entities for update/delete operations.

### Observe Overloads

| Overload | Use When |
|---|---|
| `messenger.Observe(IState<T>, keySelector)` | Single-value state |
| `messenger.Observe(IListState<T>, keySelector)` | Collection state |
| `messenger.Observe(IListState<T>, otherFeed, predicate, keySelector)` | Only refresh when related feed matches predicate |

### Fluent API

```csharp
public IState<User> CurrentUser =>
    State.Async(this, UserService.GetCurrentUser)
        .Observe(_messenger, user => user.Id);

public IListState<Person> People =>
    ListState.Async(this, PeopleService.GetAllAsync)
        .Observe(_messenger, person => person.Id);
```

### Filtered Observe

Only update when a related entity matches:

```csharp
public PeopleModel(IPeopleService peopleService, IPhoneService phoneService, IMessenger messenger)
{
    messenger.Observe(People, person => person.Id);

    // Only refresh phones when they belong to the selected person
    messenger.Observe(
        SelectedPersonPhones,
        SelectedPerson,
        (person, phone) => true,  // predicate
        phone => phone.Id);
}
```

---

## Common Mistakes

- **Missing `IMessenger` registration**: Must be a singleton — `services.AddSingleton<IMessenger, WeakReferenceMessenger>()`
- **Missing key property on records**: Entities used with `Observe` need a key for matching (see 01-records-code-generation.md)
- **Sending message before DB write completes**: Always await the repository operation before calling `messenger.Send`
- **Method returns a value**: `public string DoWork()` — won't generate a command (must return void/Task/ValueTask)

---

## Reference

- [Commands Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Advanced/Commands.html)
- [Messaging Reference](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Mvux/Advanced/Messaging.html)
