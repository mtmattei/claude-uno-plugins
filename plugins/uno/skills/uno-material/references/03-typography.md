# Typography and Font Customization

## Impact: CRITICAL
Typography is the most visible design element. Wrong style keys produce silent fallbacks to default text, making the UI look broken without any compile-time error.

---

## MD3 Type Scale

Material Design 3 defines 5 categories x 3 sizes = 15 typography styles:

| Category    | Large           | Medium            | Small            |
|-------------|-----------------|-------------------|------------------|
| **Display** | DisplayLarge    | DisplayMedium     | DisplaySmall     |
| **Headline**| HeadlineLarge   | HeadlineMedium    | HeadlineSmall    |
| **Title**   | TitleLarge      | TitleMedium       | TitleSmall       |
| **Body**    | BodyLarge       | BodyMedium        | BodySmall        |
| **Label**   | LabelLarge      | LabelMedium       | LabelSmall       |

---

## Usage on TextBlock

**Impact: CRITICAL** -- The style key is the bare `{Category}{Size}` name. No suffix.

### CORRECT
```xml
<TextBlock Text="Page Title"
           Style="{StaticResource HeadlineMedium}" />

<TextBlock Text="Body copy for paragraph text"
           Style="{StaticResource BodyLarge}" />

<TextBlock Text="Small label"
           Style="{StaticResource LabelSmall}" />
```

### WRONG
```xml
<!-- WRONG: Do NOT append "TextBlockStyle" -->
<TextBlock Text="Page Title"
           Style="{StaticResource HeadlineMediumTextBlockStyle}" />

<!-- WRONG: Do NOT use legacy "MaterialHeadline6" naming -->
<TextBlock Text="Page Title"
           Style="{StaticResource MaterialHeadline6}" />

<!-- WRONG: Do NOT invent sizes that don't exist -->
<TextBlock Text="Tiny text"
           Style="{StaticResource BodyExtraSmall}" />
```

**Pattern rule:** `{Category}{Size}` -- that is the complete key.
- CORRECT: `TitleLarge`, `BodyMedium`, `LabelSmall`
- WRONG: `TitleLargeTextBlockStyle`, `MaterialTitleLarge`, `Title1`

---

## Default Font

**Impact: MEDIUM** -- Uno Material ships with **Roboto** as the default font family. No action is needed if Roboto is acceptable.

---

## Custom Font Override

**Impact: HIGH** -- To replace Roboto with a custom font, override the three Material font family resource keys.

### Font Resource Keys

| Key                        | Weight    |
|----------------------------|-----------|
| `MaterialLightFontFamily`  | Light     |
| `MaterialMediumFontFamily` | Medium    |
| `MaterialRegularFontFamily`| Regular   |

### XAML Override (MaterialFontsOverride.xaml)

#### CORRECT
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <FontFamily x:Key="MaterialLightFontFamily">ms-appx:///Assets/Fonts/MyFont-Light.ttf</FontFamily>
    <FontFamily x:Key="MaterialMediumFontFamily">ms-appx:///Assets/Fonts/MyFont-Medium.ttf</FontFamily>
    <FontFamily x:Key="MaterialRegularFontFamily">ms-appx:///Assets/Fonts/MyFont-Regular.ttf</FontFamily>

</ResourceDictionary>
```

Then reference it in App.xaml:

```xml
<MaterialTheme x:Name="MaterialTheme"
               FontOverrideSource="ms-appx:///Styles/MaterialFontsOverride.xaml" />
```

#### WRONG
```xml
<!-- WRONG: Using wrong key names -->
<FontFamily x:Key="MaterialFontFamily">ms-appx:///Assets/Fonts/MyFont.ttf</FontFamily>

<!-- WRONG: Missing ms-appx:/// protocol -->
<FontFamily x:Key="MaterialRegularFontFamily">Assets/Fonts/MyFont-Regular.ttf</FontFamily>

<!-- WRONG: Putting overrides directly in App.xaml instead of a separate ResourceDictionary -->
```

### C# Markup Font Override

**Impact: HIGH**

#### CORRECT
```csharp
private static ResourceDictionary GetFontOverride() =>
    new ResourceDictionary()
        .Add<FontFamily>("MaterialLightFontFamily", "ms-appx:///Assets/Fonts/MyFont-Light.ttf")
        .Add<FontFamily>("MaterialMediumFontFamily", "ms-appx:///Assets/Fonts/MyFont-Medium.ttf")
        .Add<FontFamily>("MaterialRegularFontFamily", "ms-appx:///Assets/Fonts/MyFont-Regular.ttf");

// Then in App.cs:
this.Resources.MergedDictionaries.Add(GetFontOverride());
```

#### WRONG
```csharp
// WRONG: Using string instead of FontFamily type
.Add("MaterialRegularFontFamily", "ms-appx:///Assets/Fonts/MyFont-Regular.ttf")

// WRONG: Inventing key names
.Add<FontFamily>("RegularFont", "ms-appx:///Assets/Fonts/MyFont-Regular.ttf")
```

---

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Using `HeadlineMediumTextBlockStyle` instead of `HeadlineMedium` | CRITICAL | Remove the `TextBlockStyle` suffix |
| Using legacy v1 names like `MaterialHeadline6` | CRITICAL | Use MD3 names: `HeadlineMedium` |
| Font file not set to **Content** build action | HIGH | In .csproj: `<Content Include="Assets\Fonts\*.ttf" />` or set in VS properties |
| Missing `ms-appx:///` protocol prefix in font path | HIGH | Always use full `ms-appx:///Assets/Fonts/...` path |
| Overriding only one font weight key | MEDIUM | Override all three: Light, Medium, Regular |
| Font file name mismatch with path in XAML | HIGH | Verify exact filename including casing |

---

## Quick Reference: Typical Usage by Context

| Context               | Recommended Style  |
|-----------------------|-------------------|
| Hero / splash text    | DisplayLarge      |
| Page title            | HeadlineMedium    |
| Section header        | TitleLarge        |
| Card title            | TitleMedium       |
| Body paragraph        | BodyLarge         |
| Secondary text        | BodyMedium        |
| Caption / metadata    | BodySmall         |
| Button text           | LabelLarge        |
| Tab / chip label      | LabelMedium       |
| Badge / overline      | LabelSmall        |

---

## References

- [Uno Material Typography](https://platform.uno/docs/articles/external/uno.themes/doc/material-getting-started.html)
- [Material Design 3 Type Scale](https://m3.material.io/styles/typography/type-scale-tokens)
- [Uno Material Font Customization](https://platform.uno/docs/articles/external/uno.themes/doc/material-getting-started.html#customization)
