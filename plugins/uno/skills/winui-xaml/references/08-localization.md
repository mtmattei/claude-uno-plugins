# Localization Skills - WinUI 3 XAML

## LOC-001: Use x:Uid for All User-Visible Text

**Rule**: Set `x:Uid` on every element with user-visible text. Define localized values in `.resw` resource files.

**Why**: x:Uid enables localization without code changes. Translators work with resource files, not XAML. Text is resolved at runtime based on user's language settings.

**Example (Correct)**:
```xml
<!-- XAML with x:Uid -->
<StackPanel>
    <TextBlock x:Uid="WelcomePage_Title"/>
    <TextBlock x:Uid="WelcomePage_Subtitle"/>
    <Button x:Uid="WelcomePage_GetStartedButton"/>
</StackPanel>
```

**Resource file (Strings/en-US/Resources.resw)**:
| Name | Value |
|------|-------|
| WelcomePage_Title.Text | Welcome to MyApp |
| WelcomePage_Subtitle.Text | Get started in minutes |
| WelcomePage_GetStartedButton.Content | Get Started |

**Resource file (Strings/es-ES/Resources.resw)**:
| Name | Value |
|------|-------|
| WelcomePage_Title.Text | Bienvenido a MyApp |
| WelcomePage_Subtitle.Text | Empieza en minutos |
| WelcomePage_GetStartedButton.Content | Empezar |

**x:Uid Naming Convention**: `PageName_ElementPurpose` or `ControlType_Purpose`

**Common Mistakes**:
```xml
<!-- WRONG: Hardcoded text cannot be localized -->
<TextBlock Text="Welcome to MyApp"/>
<Button Content="Get Started"/>
```

**Uno Platform Notes**: x:Uid is fully supported. Resource files are resolved per platform conventions.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/globalizing/localize-strings-ui-manifest

---

## LOC-002: Set Multiple Properties via x:Uid

**Rule**: Use x:Uid to set multiple properties including Text, Content, AutomationProperties.Name, Header, PlaceholderText, and ToolTipService.ToolTip.

**Why**: A single x:Uid can localize multiple properties of an element. This ensures consistency and reduces resource file entries.

**Example (Correct)**:
```xml
<TextBox x:Uid="LoginPage_EmailInput"/>
```

**Resources.resw**:
| Name | Value |
|------|-------|
| LoginPage_EmailInput.Header | Email address |
| LoginPage_EmailInput.PlaceholderText | name@example.com |
| LoginPage_EmailInput.[AutomationProperties.Name] | Email address input |
| LoginPage_EmailInput.[ToolTipService.ToolTip] | Enter your email address |

**Button with accessible name**:
```xml
<Button x:Uid="MainPage_CloseButton">
    <FontIcon Glyph="&#xE711;"/>
</Button>
```

| Name | Value |
|------|-------|
| MainPage_CloseButton.[AutomationProperties.Name] | Close |
| MainPage_CloseButton.[ToolTipService.ToolTip] | Close this panel |

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/globalizing/localize-strings-ui-manifest#refer-to-a-string-resource-identifier-from-xaml

---

## LOC-003: Access Strings in Code-Behind

**Rule**: Use `ResourceLoader` to access localized strings in C# code.

**Why**: Some strings are needed in code (error messages, dynamic content, notifications). ResourceLoader provides runtime access to .resw resources.

**Example (Correct)**:
```csharp
using Windows.ApplicationModel.Resources;

public sealed partial class MainPage : Page
{
    private readonly ResourceLoader _resourceLoader;

    public MainPage()
    {
        InitializeComponent();
        _resourceLoader = ResourceLoader.GetForViewIndependentUse();
    }

    private void ShowError()
    {
        var message = _resourceLoader.GetString("ErrorMessages_NetworkError");
        ErrorText.Text = message;
    }

    private string GetFormattedCount(int count)
    {
        var format = _resourceLoader.GetString("ItemCount_Format"); // "{0} items"
        return string.Format(format, count);
    }
}
```

**Resources.resw**:
| Name | Value |
|------|-------|
| ErrorMessages_NetworkError | Unable to connect. Check your internet connection. |
| ItemCount_Format | {0} items |

**Common Mistakes**:
```csharp
// WRONG: Hardcoded strings in code
private void ShowError()
{
    ErrorText.Text = "Unable to connect. Check your internet connection.";
}
```

**Uno Platform Notes**: With Uno.Extensions.Localization, use `IStringLocalizer` for dependency-injected string resolution.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/globalizing/localize-strings-ui-manifest#load-strings-from-code

---

## LOC-004: Organize Resources by Feature

**Rule**: Structure resource keys hierarchically: `FeatureName_ElementName.Property` or `PageName/ControlName.Property`.

**Why**: Consistent naming prevents key collisions, makes resources discoverable, and helps translators understand context.

**Example (Correct)**:
```
Strings/
  en-US/
    Resources.resw       # Shared strings
    LoginPage.resw       # Login-specific (optional split)
  es-ES/
    Resources.resw
    LoginPage.resw
```

**Naming conventions**:
```
# Page-specific
LoginPage_Title.Text
LoginPage_EmailInput.Header
LoginPage_PasswordInput.Header
LoginPage_SignInButton.Content

# Shared/common
Common_Cancel.Content
Common_Save.Content
Common_Delete.Content

# Errors
Errors_NetworkUnavailable
Errors_InvalidEmail
Errors_RequiredField

# Validation messages
Validation_EmailFormat
Validation_PasswordStrength
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/globalizing/prepare-your-app-for-localization

---

## LOC-005: Handle Right-to-Left Languages

**Rule**: Use `FlowDirection="{ThemeResource LayoutFlowDirection}"` and avoid hardcoded margins/paddings that assume LTR.

**Why**: RTL languages (Arabic, Hebrew) require mirrored layouts. Using theme resources and symmetric layouts ensures proper RTL support.

**Example (Correct)**:
```xml
<!-- Page-level flow direction -->
<Page FlowDirection="{ThemeResource LayoutFlowDirection}">
    <Grid>
        <!-- Symmetric padding -->
        <StackPanel Padding="16">
            <TextBlock Text="{x:Bind Title}"/>
        </StackPanel>
    </Grid>
</Page>

<!-- Icons that should NOT flip -->
<FontIcon Glyph="&#xE768;" FlowDirection="LeftToRight"/>
```

**Common Mistakes**:
```xml
<!-- WRONG: Asymmetric margin assumes LTR -->
<Grid Margin="16,0,0,0">  <!-- Left margin only -->

<!-- WRONG: Hardcoded alignment assumes LTR -->
<TextBlock HorizontalAlignment="Left"/>
```

**Uno Platform Notes**: FlowDirection is supported on all platforms. Test RTL layout on each target platform.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/globalizing/design-for-bidi-text

---

## LOC-006: Support Runtime Language Switching

**Rule**: For apps that support runtime language switching, use proper resource invalidation and consider page reload.

**Why**: Some apps need to change language without restart. This requires resource cache invalidation and UI refresh.

**Example (Correct with Uno.Extensions)**:
```csharp
using Uno.Extensions.Localization;

public class SettingsViewModel
{
    private readonly ILocalizationService _localization;

    public SettingsViewModel(ILocalizationService localization)
    {
        _localization = localization;
    }

    public async Task ChangeLanguageAsync(string cultureCode)
    {
        await _localization.SetCurrentCultureAsync(new CultureInfo(cultureCode));
        // UI will update automatically with Uno.Extensions
    }
}
```

**Standard WinUI approach**:
```csharp
public void ChangeLanguage(string languageCode)
{
    Windows.Globalization.ApplicationLanguages.PrimaryLanguageOverride = languageCode;

    // Reload current page to apply new resources
    var frame = Window.Current.Content as Frame;
    frame.Navigate(frame.Content.GetType());
    frame.GoBack();
}
```

**Reference**: https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Localization/LocalizationOverview.html

---

## LOC-007: Localize Dates, Numbers, and Currency

**Rule**: Use `DateTimeFormatter`, `NumberFormatter`, and culture-aware formatting for dates, numbers, and currency.

**Why**: Date/number formats vary by locale (MM/DD vs DD/MM, comma vs period decimal). Using formatters ensures culturally appropriate display.

**Example (Correct)**:
```csharp
using Windows.Globalization.DateTimeFormatting;
using Windows.Globalization.NumberFormatting;

// Date formatting
var formatter = new DateTimeFormatter("shortdate");
var formattedDate = formatter.Format(DateTime.Now);
// en-US: "1/22/2026"
// de-DE: "22.01.2026"

// Number formatting
var numberFormatter = new DecimalFormatter
{
    IsDecimalPointAlwaysDisplayed = true,
    FractionDigits = 2
};
var formattedNumber = numberFormatter.FormatDouble(1234.5);
// en-US: "1,234.50"
// de-DE: "1.234,50"

// Currency formatting
var currencyFormatter = new CurrencyFormatter("USD");
var formattedCurrency = currencyFormatter.FormatDouble(99.99);
// en-US: "$99.99"
```

**Common Mistakes**:
```csharp
// WRONG: Hardcoded format
var dateText = date.ToString("MM/dd/yyyy"); // Wrong for non-US locales

// WRONG: String concatenation for currency
var priceText = "$" + price.ToString(); // Wrong currency symbol for non-US
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/globalizing/use-global-ready-formats
