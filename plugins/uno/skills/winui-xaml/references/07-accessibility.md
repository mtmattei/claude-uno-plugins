# Accessibility Skills - WinUI 3 XAML

## A11Y-001: Set AutomationProperties.Name for Interactive Elements

**Rule**: Set `AutomationProperties.Name` on all buttons, links, and interactive elements that don't have visible text labels.

**Why**: Screen readers use AutomationProperties.Name to announce elements. Without it, users hear "button" instead of "Submit form" or "Close dialog".

**Example (Correct)**:
```xml
<!-- Icon-only button needs Name -->
<Button AutomationProperties.Name="Close dialog"
        Click="CloseButton_Click">
    <FontIcon Glyph="&#xE711;"/>
</Button>

<!-- Image button needs Name -->
<Button AutomationProperties.Name="Add to favorites">
    <Image Source="/Assets/heart.png" Width="24" Height="24"/>
</Button>

<!-- Button with text - Name is inferred from Content -->
<Button Content="Submit"/>  <!-- AutomationProperties.Name = "Submit" -->
```

**Common Mistakes**:
```xml
<!-- WRONG: No accessible name -->
<Button Click="CloseButton_Click">
    <FontIcon Glyph="&#xE711;"/>
</Button>
<!-- Screen reader announces: "Button" -->

<!-- WRONG: Redundant "button" in name -->
<Button AutomationProperties.Name="Close button">
    <FontIcon Glyph="&#xE711;"/>
</Button>
<!-- Screen reader announces: "Close button button" -->
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/basic-accessibility-information

---

## A11Y-002: Use AutomationProperties.LabeledBy for Form Fields

**Rule**: Associate labels with form fields using `AutomationProperties.LabeledBy` or place labels in `Header` property.

**Why**: Screen readers need to know which label describes which input. Without association, users don't know what information to enter.

**Example (Correct)**:
```xml
<!-- Option 1: Use Header property (preferred) -->
<TextBox Header="Email address"
         PlaceholderText="name@example.com"/>

<!-- Option 2: Use LabeledBy -->
<StackPanel>
    <TextBlock x:Name="NameLabel" Text="Full name"/>
    <TextBox AutomationProperties.LabeledBy="{Binding ElementName=NameLabel}"/>
</StackPanel>

<!-- Option 3: Combined label and input -->
<StackPanel>
    <TextBlock Text="Phone number" x:Name="PhoneLabel"/>
    <TextBox AutomationProperties.LabeledBy="{Binding ElementName=PhoneLabel}"
             InputScope="TelephoneNumber"/>
</StackPanel>
```

**Common Mistakes**:
```xml
<!-- WRONG: Label not associated with input -->
<StackPanel>
    <TextBlock Text="Email"/>
    <TextBox/>  <!-- Screen reader doesn't know this is for email -->
</StackPanel>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/accessible-text-requirements

---

## A11Y-003: Ensure 4.5:1 Color Contrast Ratio

**Rule**: Maintain minimum 4.5:1 contrast ratio for body text, 3:1 for large text (18pt+) and UI components.

**Why**: Low contrast makes text difficult to read for users with visual impairments. WCAG 2.1 AA compliance requires these ratios.

**Example (Correct)**:
```xml
<!-- Use semantic theme resources that maintain contrast -->
<TextBlock Text="Important information"
           Foreground="{ThemeResource TextFillColorPrimaryBrush}"/>

<TextBlock Text="Secondary text"
           Foreground="{ThemeResource TextFillColorSecondaryBrush}"/>

<!-- Error text with sufficient contrast -->
<TextBlock Text="Invalid email address"
           Foreground="{ThemeResource SystemFillColorCriticalBrush}"/>
```

**Common Mistakes**:
```xml
<!-- WRONG: Light gray on white - poor contrast -->
<TextBlock Text="Hint text"
           Foreground="#CCCCCC"/>  <!-- ~1.6:1 contrast - fails -->

<!-- WRONG: Hardcoded colors that may fail in themes -->
<TextBlock Text="Important"
           Foreground="#888888"/>
```

**Uno Platform Notes**: Use theme resources that adapt to light/dark modes. Test on all platforms.

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/accessible-text-requirements

---

## A11Y-004: Set Minimum Touch Target Size

**Rule**: Interactive elements must have minimum 44x44 pixels (or 48x48 dp) touch target size.

**Why**: Small touch targets are difficult for users with motor impairments or tremors. They also frustrate all users on touch devices.

**Example (Correct)**:
```xml
<!-- Explicit minimum size -->
<Button Content="X" MinWidth="44" MinHeight="44"
        AutomationProperties.Name="Close"/>

<!-- Icon button with padding for touch target -->
<Button Padding="12" MinWidth="44" MinHeight="44">
    <FontIcon Glyph="&#xE74D;" FontSize="16"/>
</Button>

<!-- Small visual with larger hit area -->
<Button Width="44" Height="44" Background="Transparent">
    <Ellipse Width="12" Height="12" Fill="Red"/>
</Button>
```

**Common Mistakes**:
```xml
<!-- WRONG: Too small for touch -->
<Button Width="24" Height="24" Content="X"/>

<!-- WRONG: List items without adequate spacing -->
<ListView.ItemTemplate>
    <DataTemplate>
        <TextBlock Text="{x:Bind Name}" Padding="4"/>  <!-- Too small -->
    </DataTemplate>
</ListView.ItemTemplate>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/guidelines-for-targeting

---

## A11Y-005: Support Keyboard Navigation

**Rule**: Ensure all interactive elements are keyboard accessible. Set `TabIndex` for custom focus order. Handle Enter/Space for activation.

**Why**: Users who can't use a mouse rely on keyboard navigation. All functionality must be accessible via keyboard.

**Example (Correct)**:
```xml
<!-- Logical tab order -->
<StackPanel>
    <TextBox Header="First name" TabIndex="1"/>
    <TextBox Header="Last name" TabIndex="2"/>
    <TextBox Header="Email" TabIndex="3"/>
    <Button Content="Submit" TabIndex="4"/>
</StackPanel>

<!-- Custom control with keyboard support -->
<Grid x:Name="ClickableCard"
      IsTabStop="True"
      TabFocusNavigation="Once"
      KeyDown="Card_KeyDown"
      PointerPressed="Card_PointerPressed">
    <FocusVisualManager.FocusVisualMargin>-3</FocusVisualManager.FocusVisualMargin>
    <!-- Card content -->
</Grid>
```

```csharp
private void Card_KeyDown(object sender, KeyRoutedEventArgs e)
{
    if (e.Key == VirtualKey.Enter || e.Key == VirtualKey.Space)
    {
        e.Handled = true;
        ActivateCard();
    }
}
```

**Common Mistakes**:
```xml
<!-- WRONG: Not keyboard accessible -->
<Grid PointerPressed="Card_Click">
    <!-- No IsTabStop, no KeyDown handler -->
</Grid>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/input/keyboard-interactions

---

## A11Y-006: Provide Text Alternatives for Images

**Rule**: Set `AutomationProperties.Name` on meaningful images. Use empty string for decorative images.

**Why**: Screen readers need descriptions of meaningful images. Decorative images should be skipped.

**Example (Correct)**:
```xml
<!-- Meaningful image - describe content -->
<Image Source="/Assets/product-photo.jpg"
       AutomationProperties.Name="Blue running shoes, side view"/>

<!-- Chart/graph - describe data -->
<Image Source="{x:Bind ChartImageUrl}"
       AutomationProperties.Name="{x:Bind ChartDescription}"/>

<!-- Decorative image - empty name to skip -->
<Image Source="/Assets/decorative-border.png"
       AutomationProperties.AccessibilityView="Raw"/>

<!-- Image with adjacent text description -->
<StackPanel>
    <Image Source="{x:Bind Photo}"
           AutomationProperties.AccessibilityView="Raw"/>
    <TextBlock Text="{x:Bind PhotoCaption}"/>  <!-- Provides description -->
</StackPanel>
```

**Common Mistakes**:
```xml
<!-- WRONG: No accessible name for meaningful image -->
<Image Source="/Assets/warning-icon.png"/>

<!-- WRONG: Redundant description -->
<Image Source="/Assets/photo.jpg"
       AutomationProperties.Name="Image of a photo"/>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/images-accessibility

---

## A11Y-007: Use Live Regions for Dynamic Content

**Rule**: Set `AutomationProperties.LiveSetting` on elements that update dynamically to announce changes to screen readers.

**Why**: Screen readers don't automatically announce content changes. LiveSetting ensures users are notified of important updates.

**Example (Correct)**:
```xml
<!-- Status messages that should be announced -->
<TextBlock x:Name="StatusMessage"
           AutomationProperties.LiveSetting="Assertive"/>

<!-- Polite updates (less urgent) -->
<TextBlock x:Name="ItemCount"
           Text="{x:Bind ViewModel.ItemCountText, Mode=OneWay}"
           AutomationProperties.LiveSetting="Polite"/>

<!-- Search results count -->
<TextBlock AutomationProperties.LiveSetting="Polite">
    <Run Text="{x:Bind ViewModel.ResultCount, Mode=OneWay}"/>
    <Run Text=" results found"/>
</TextBlock>
```

```csharp
// Announce status changes
private void ShowError(string message)
{
    StatusMessage.Text = message;
    // LiveSetting=Assertive ensures immediate announcement
}
```

**LiveSetting Values**:
- `Off`: No announcement (default)
- `Polite`: Announce when user is idle
- `Assertive`: Announce immediately, interrupting

**Reference**: https://learn.microsoft.com/en-us/uwp/api/windows.ui.xaml.automation.peers.automationlivesetting

---

## A11Y-008: Support High Contrast Mode

**Rule**: Use theme resources that adapt to high contrast. Avoid hardcoded colors. Test in High Contrast mode.

**Why**: High Contrast mode is essential for users with visual impairments. Hardcoded colors become invisible or unreadable.

**Example (Correct)**:
```xml
<!-- Use theme resources -->
<Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
        BorderBrush="{ThemeResource CardStrokeColorDefaultBrush}">
    <TextBlock Text="Card content"
               Foreground="{ThemeResource TextFillColorPrimaryBrush}"/>
</Border>

<!-- Resource with high contrast override -->
<Page.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="HighContrast">
                <SolidColorBrush x:Key="MyAccentBrush"
                                 Color="{ThemeResource SystemColorHighlightColor}"/>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Page.Resources>
```

**Common Mistakes**:
```xml
<!-- WRONG: Hardcoded colors fail in high contrast -->
<Border Background="#1E1E1E">
    <TextBlock Foreground="#FFFFFF"/>
</Border>
```

**Reference**: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/high-contrast-themes
