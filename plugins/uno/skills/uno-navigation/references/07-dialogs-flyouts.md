# Dialogs and Flyouts

How to show dialogs, flyouts, message dialogs, and content dialogs using Uno Platform's navigation system with the `!` (dialog) qualifier.

---

## Dialog Qualifier

**Impact**: CRITICAL — The `!` qualifier is what makes navigation open a dialog instead of navigating to a page.

The same view (Page or ContentDialog) can be shown as a page or as a dialog depending on the qualifier. Adding `!` to the route turns it into a dialog.

### From XAML

```xml
<!-- Opens SamplePage as a flyout dialog -->
<Button Content="Show Sample"
        uen:Navigation.Request="!Sample" />
```

### From Code

```csharp
// Opens SamplePage as a flyout dialog
await navigator.NavigateViewAsync<SamplePage>(this, qualifier: Qualifiers.Dialog);

// Or by route name with ! prefix
await navigator.NavigateRouteAsync(this, "!Sample");
```

### How the View Type Determines Dialog Style

| View Type | Dialog Behavior |
|---|---|
| `Page` | Shown as a Flyout overlay |
| `ContentDialog` | Shown as a Modal dialog |

---

## ContentDialog (Modal)

**Impact**: HIGH — For modal prompts that require user action before continuing.

To show a modal dialog, change the view from a `Page` to a `ContentDialog` and register it normally.

### View

```xml
<!-- SampleDialog.xaml -->
<ContentDialog x:Class="MyApp.Presentation.SampleDialog"
               xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
               xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
               Title="Confirm Action"
               PrimaryButtonText="Confirm"
               SecondaryButtonText="Cancel"
               CloseButtonText="Close">
    <StackPanel Spacing="12">
        <TextBlock Text="Are you sure you want to proceed?" />
    </StackPanel>
</ContentDialog>
```

### Registration

```csharp
views.Register(
    new ViewMap<SampleDialog, SampleDialogViewModel>()
);

routes.Register(
    new RouteMap("Sample", View: views.FindByViewModel<SampleDialogViewModel>())
);
```

### Show It

```csharp
// From code — qualifier makes it a dialog
await navigator.NavigateViewAsync<SampleDialog>(this, qualifier: Qualifiers.Dialog);
```

```xml
<!-- From XAML — ! prefix makes it a dialog -->
<Button Content="Show Dialog"
        uen:Navigation.Request="!Sample" />
```

```csharp
// WRONG: navigating without ! qualifier — opens as a page, not a dialog
await navigator.NavigateRouteAsync(this, "Sample");
```

---

## MessageDialog

**Impact**: HIGH — Quick alerts and confirmations without creating a custom view.

### Simple Message

```csharp
var navigator = this.Navigator();

// Simple alert with title and content
await navigator.ShowMessageDialogAsync(this,
    title: "Error",
    content: "Something went wrong. Please try again.");
```

### Message with Button Choices

```csharp
var result = await navigator.ShowMessageDialogAsync(this,
    title: "Delete Item",
    content: "This action cannot be undone.",
    buttons: new[]
    {
        new DialogAction("Delete"),
        new DialogAction("Cancel")
    });

// result is the index of the button pressed (0 = Delete, 1 = Cancel)
```

### Reusable MessageDialogViewMap

Register a reusable message dialog configuration so you don't repeat yourself:

```csharp
views.Register(
    new MessageDialogViewMap(
        Title: "Confirm",
        Content: "Are you sure?",
        Buttons: new[]
        {
            new DialogAction("OK"),
            new DialogAction("Cancel")
        }
    )
);
```

### LocalizableMessageDialogViewMap

For localized message dialogs, use the localizable variant that resolves strings from resources:

```csharp
views.Register(
    new LocalizableMessageDialogViewMap(
        Title: localizer => localizer["Dialog_Confirm_Title"],
        Content: localizer => localizer["Dialog_Confirm_Content"],
        Buttons: new[]
        {
            new LocalizableDialogAction(Label: localizer => localizer["Dialog_OK"]),
            new LocalizableDialogAction(Label: localizer => localizer["Dialog_Cancel"])
        }
    )
);
```

---

## Custom Dialog Pattern (Chefs Reference)

**Impact**: MEDIUM — A reusable generic dialog pattern from the Uno Chefs sample app.

### DialogInfo Record

```csharp
public record DialogInfo(
    string Title,
    string Content,
    string AcceptButtonText = "OK",
    string? CancelButtonText = null);
```

### GenericDialog View

```xml
<!-- GenericDialog.xaml -->
<ContentDialog x:Class="MyApp.Presentation.GenericDialog"
               xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
               xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
               Title="{Binding Title}"
               PrimaryButtonText="{Binding AcceptButtonText}"
               SecondaryButtonText="{Binding CancelButtonText}">
    <TextBlock Text="{Binding Content}"
               TextWrapping="Wrap" />
</ContentDialog>
```

### Registration with DataMap

```csharp
views.Register(
    new DataViewMap<GenericDialog, GenericDialogViewModel, DialogInfo>()
);

routes.Register(
    new RouteMap("GenericDialog", View: views.FindByViewModel<GenericDialogViewModel>())
);
```

### Usage

```csharp
// Show reusable dialog with data
await navigator.NavigateDataAsync(this,
    data: new DialogInfo(
        Title: "Delete Recipe",
        Content: "Are you sure you want to delete this recipe?",
        AcceptButtonText: "Delete",
        CancelButtonText: "Cancel"),
    qualifier: Qualifiers.Dialog);
```

---

## Dialog Results

**Impact**: HIGH — How to get data back from a dialog.

### NavigateBackWithResultAsync

The dialog view (or its ViewModel) calls `NavigateBackWithResultAsync` to return data to the caller.

```csharp
// In the dialog's ViewModel or code-behind
public async ValueTask Confirm(CancellationToken ct)
{
    var navigator = this.Navigator();
    await navigator.NavigateBackWithResultAsync(this, data: selectedItem);
}
```

### Receiving the Result

Use `GetDataAsync` on the navigation response to await the dialog's result:

```csharp
// Show dialog and wait for result
var response = await navigator.NavigateViewAsync<PickerDialog>(this,
    qualifier: Qualifiers.Dialog);

// Wait for the dialog to close and return data
var result = await response.GetDataAsync<PickedItem>();
if (result is not null)
{
    // User made a selection
    await ProcessSelection(result);
}
```

### ResultDataViewMap Registration

For typed round-trip data (send data in, get result back), use `ResultDataViewMap`:

```csharp
views.Register(
    new ResultDataViewMap<PickerDialog, PickerViewModel, PickerInput, PickedItem>()
);
```

```csharp
// CORRECT: using ResultDataViewMap — navigation knows both input and output types
var response = await navigator.NavigateDataAsync(this,
    data: new PickerInput(Options: availableItems),
    qualifier: Qualifiers.Dialog);
var picked = await response.GetDataAsync<PickedItem>();
```

```csharp
// WRONG: using DataViewMap for round-trip — result type not registered
views.Register(
    new DataViewMap<PickerDialog, PickerViewModel, PickerInput>()
);
// GetDataAsync<PickedItem>() may fail or return null
```

---

## Common Mistakes

- **Missing `!` qualifier**: Without `!`, navigation opens a page instead of a dialog. This is the most common dialog bug.
- **Using `NavigateRouteAsync` without `!` prefix**: `NavigateRouteAsync(this, "Sample")` navigates to a page. Must be `NavigateRouteAsync(this, "!Sample")` or use `qualifier: Qualifiers.Dialog`.
- **ContentDialog without PrimaryButtonText**: Dialog appears but has no way to close except programmatically. Always set at least one button text.
- **Forgetting to `await` GetDataAsync**: The result is only available after the dialog closes. Not awaiting means you get null immediately.
- **Using DataViewMap instead of ResultDataViewMap for round-trip**: The navigation system needs ResultDataViewMap to know the return type. DataViewMap only handles input data.
- **Not calling NavigateBackWithResultAsync**: If the dialog ViewModel just calls `NavigateBackAsync`, no result data is returned to the caller.

---

## Reference

- [Navigation Dialogs](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/Dialogs.html)
- [MessageDialog](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/MessageDialog.html)
- [Navigation Qualifiers](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/Qualifiers.html)
- [ResultDataViewMap](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/RouteMap.html)
