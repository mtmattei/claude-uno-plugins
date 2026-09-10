# Control Styles

## Impact: CRITICAL
Control styles are the primary way to apply Material Design appearance to UI controls. Using the wrong style key or namespace produces silent failures where the control renders with the default WinUI chrome instead of Material Design.

---

## Button Styles

**Impact: CRITICAL**

| Style Key                  | Appearance                              | Default? |
|----------------------------|-----------------------------------------|----------|
| `FilledButtonStyle`        | Solid primary-color background          | Yes      |
| `ElevatedButtonStyle`      | Surface color with shadow/elevation     | No       |
| `FilledTonalButtonStyle`   | Secondary container color background    | No       |
| `OutlinedButtonStyle`      | Transparent with border outline         | No       |
| `TextButtonStyle`          | Text only, no background or border      | No       |
| `IconButtonStyle`          | Icon-only circular button               | No       |

### CORRECT
```xml
<Button Content="Save"
        Style="{StaticResource FilledButtonStyle}" />

<Button Content="Cancel"
        Style="{StaticResource OutlinedButtonStyle}" />

<Button Content="Details"
        Style="{StaticResource TextButtonStyle}" />

<Button Style="{StaticResource IconButtonStyle}">
    <SymbolIcon Symbol="Edit" />
</Button>
```

### WRONG
```xml
<!-- WRONG: No such style exists -->
<Button Content="Save"
        Style="{StaticResource MaterialFilledButtonStyle}" />

<!-- WRONG: "Contained" is MD2 naming, not MD3 -->
<Button Content="Save"
        Style="{StaticResource ContainedButtonStyle}" />

<!-- WRONG: Putting text Content on an IconButtonStyle -->
<Button Content="Edit"
        Style="{StaticResource IconButtonStyle}" />
```

---

## ToggleButton Styles

**Impact: HIGH**

Use `IconToggleButtonStyle` with `ControlExtensions.AlternateContent` for checked/unchecked icon states.

### CORRECT
```xml
xmlns:ut="using:Uno.Themes"

<ToggleButton Style="{StaticResource IconToggleButtonStyle}"
              ut:ControlExtensions.AlternateContent="{ut:FontIcon Glyph='&#xE00B;'}">
    <FontIcon Glyph="&#xE006;" />
</ToggleButton>
```

The default content shows when **unchecked**. The `AlternateContent` shows when **checked**.

### WRONG
```xml
<!-- WRONG: Using um: namespace (v4 and earlier) instead of ut: (v5+) -->
<ToggleButton Style="{StaticResource IconToggleButtonStyle}"
              um:ControlExtensions.AlternateContent="{um:FontIcon Glyph='&#xE00B;'}">
    <FontIcon Glyph="&#xE006;" />
</ToggleButton>
```

---

## TextBox Styles

**Impact: HIGH**

| Style Key               | Appearance                         | Default? |
|--------------------------|------------------------------------|----------|
| `OutlinedTextBoxStyle`   | Border outline, floating label     | Yes      |
| `FilledTextBoxStyle`     | Filled background, underline       | No       |

### CORRECT
```xml
<TextBox Style="{StaticResource FilledTextBoxStyle}"
         Header="Email"
         PlaceholderText="Enter your email" />

<TextBox Style="{StaticResource OutlinedTextBoxStyle}"
         Header="Name"
         PlaceholderText="Enter your name" />
```

### Header and PlaceholderText Behavior (v5.1+)

- **Header**: Acts as the floating label. Animates up above the input when focused or when text is present.
- **PlaceholderText**: Shown inside the input area only when the field is empty AND unfocused. Disappears on focus.

Both can be used together -- Header provides the persistent label, PlaceholderText provides a hint.

### WRONG
```xml
<!-- WRONG: Using only PlaceholderText as the label (it disappears on focus) -->
<TextBox Style="{StaticResource OutlinedTextBoxStyle}"
         PlaceholderText="Email" />

<!-- WRONG: Non-existent style name -->
<TextBox Style="{StaticResource MaterialTextBoxStyle}" />
```

---

## PasswordBox Styles

**Impact: HIGH**

| Style Key                    | Appearance                         | Default? |
|------------------------------|------------------------------------|----------|
| `OutlinedPasswordBoxStyle`   | Border outline, floating label     | Yes      |
| `FilledPasswordBoxStyle`     | Filled background, underline       | No       |

### CORRECT
```xml
<PasswordBox Style="{StaticResource FilledPasswordBoxStyle}"
             Header="Password"
             PlaceholderText="Enter password" />
```

---

## Floating Action Button (FAB) Styles

**Impact: HIGH**

| Style Key              | Description                              |
|------------------------|------------------------------------------|
| `FabStyle`             | Primary FAB (default)                    |
| `SmallFabStyle`        | Compact FAB                              |
| `LargeFabStyle`        | Large FAB                                |
| `SurfaceFabStyle`      | Surface-colored FAB                      |
| `SecondaryFabStyle`    | Secondary-colored FAB                    |
| `TertiaryFabStyle`     | Tertiary-colored FAB                     |

FABs are applied to a standard `Button` control combined with `ControlExtensions.Icon`.

### CORRECT
```xml
xmlns:ut="using:Uno.Themes"

<!-- Standard FAB with icon and label -->
<Button Content="New Item"
        Style="{StaticResource FabStyle}">
    <ut:ControlExtensions.Icon>
        <SymbolIcon Symbol="Add" />
    </ut:ControlExtensions.Icon>
</Button>

<!-- Icon-only FAB (no Content) -->
<Button Style="{StaticResource SmallFabStyle}">
    <ut:ControlExtensions.Icon>
        <SymbolIcon Symbol="Add" />
    </ut:ControlExtensions.Icon>
</Button>

<!-- Large tertiary FAB -->
<Button Content="Compose"
        Style="{StaticResource LargeFabStyle}">
    <ut:ControlExtensions.Icon>
        <FontIcon Glyph="&#xE104;" />
    </ut:ControlExtensions.Icon>
</Button>
```

### WRONG
```xml
<!-- WRONG: Using FAB style without ControlExtensions.Icon -->
<Button Content="Add"
        Style="{StaticResource FabStyle}" />

<!-- WRONG: Using old um: namespace -->
<Button Style="{StaticResource FabStyle}">
    <um:ControlExtensions.Icon>
        <SymbolIcon Symbol="Add" />
    </um:ControlExtensions.Icon>
</Button>

<!-- WRONG: Inventing style names -->
<Button Style="{StaticResource FloatingActionButtonStyle}" />
```

---

## Control Extensions

**Impact: HIGH**

Control extensions live in the `Uno.Themes` namespace (since v5). Always use the `ut:` prefix.

### Namespace Declaration

#### CORRECT
```xml
xmlns:ut="using:Uno.Themes"
```

#### WRONG
```xml
<!-- WRONG: Old namespace from v4 and earlier -->
xmlns:um="using:Uno.Material"
```

### ControlExtensions.Icon

Adds an `IconElement` (SymbolIcon, FontIcon, PathIcon, BitmapIcon) to supported controls.

```xml
<Button Content="Search"
        Style="{StaticResource OutlinedButtonStyle}">
    <ut:ControlExtensions.Icon>
        <SymbolIcon Symbol="Find" />
    </ut:ControlExtensions.Icon>
</Button>

<TextBox Style="{StaticResource OutlinedTextBoxStyle}"
         Header="Search">
    <ut:ControlExtensions.Icon>
        <FontIcon Glyph="&#xE094;" />
    </ut:ControlExtensions.Icon>
</TextBox>
```

### ControlExtensions.Elevation

Applies Material Design shadow elevation (levels 0-5) on supported controls.

```xml
<Border Background="{ThemeResource SurfaceBrush}"
        ut:ControlExtensions.Elevation="2"
        CornerRadius="12"
        Padding="16">
    <TextBlock Text="Elevated card" Style="{StaticResource BodyLarge}" />
</Border>
```

| Level | Usage Context                |
|-------|------------------------------|
| 0     | Flat / no shadow             |
| 1     | Cards, buttons at rest       |
| 2     | Raised cards, FABs at rest   |
| 3     | Navigation drawers           |
| 4     | App bars                     |
| 5     | Dialogs, modals              |

### ControlExtensions.AlternateContent

Toggle content for ToggleButton checked/unchecked states (shown in ToggleButton section above).

---

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Using `um:` namespace for ControlExtensions | CRITICAL | Change to `ut:` (`using:Uno.Themes`) for v5+ |
| Using `ContainedButtonStyle` (MD2 name) | CRITICAL | Use `FilledButtonStyle` (MD3 name) |
| FAB without `ControlExtensions.Icon` | HIGH | Always set `ut:ControlExtensions.Icon` on FAB buttons |
| Using `MaterialFilledButtonStyle` prefix | HIGH | Drop the `Material` prefix: just `FilledButtonStyle` |
| Elevation value > 5 | MEDIUM | Max elevation level is 5 |
| Using PlaceholderText alone as label | MEDIUM | Use `Header` for the persistent floating label |
| Putting text Content on IconButtonStyle | MEDIUM | Use icon child element, not text Content |

---

## References

- [Uno Material Button Styles](https://platform.uno/docs/articles/external/uno.themes/doc/material-button-styles.html)
- [Uno Material TextBox Styles](https://platform.uno/docs/articles/external/uno.themes/doc/material-textbox-styles.html)
- [Uno Material FAB Styles](https://platform.uno/docs/articles/external/uno.themes/doc/material-fab.html)
- [Uno Themes Control Extensions](https://platform.uno/docs/articles/external/uno.themes/doc/control-extensions.html)
- [Material Design 3 Components](https://m3.material.io/components)
