# User Interface Wiki — Uno Platform / WinUI 3 Adaptation

**Version 1.0.0**
Adapted from raphael-salaja/userinterface-wiki
April 2026

> **Note:**
> This document is for agents and LLMs reviewing, generating, or refactoring
> XAML/C# UI code for WinUI 3 and Uno Platform. It adapts the web-focused
> userinterface-wiki rules to XAML equivalents. Universal rules (Animation
> Principles, Timing Functions, Laws of UX) remain in the original skill.

---

## Abstract

UI/UX best practices adapted from the web-focused userinterface-wiki for WinUI 3, XAML, and Uno Platform cross-platform applications. Contains 61 rules across 8 categories. Each rule includes incorrect vs. correct XAML/C# code examples.

---

## Table of Contents

1. [XAML Typography](#1-xaml-typography) — **HIGH**
2. [Transitions & Exit Animations](#2-transitions--exit-animations) — **HIGH**
3. [Visual States & Overlays](#3-visual-states--overlays) — **HIGH**
4. [Layout Animation](#4-layout-animation) — **MEDIUM**
5. [Data Prefetching](#5-data-prefetching) — **MEDIUM**
6. [Animated Icons](#6-animated-icons) — **MEDIUM**
7. [XAML Audio](#7-xaml-audio) — **MEDIUM**
8. [Visual Design XAML](#8-visual-design-xaml) — **HIGH**

---

## 1. XAML Typography

**Impact:** HIGH
**Description:** XAML Typography attached properties and text formatting. Numeric alignment, numeral styles, slashed zeros, character spacing, text trimming, and font configuration for WinUI/Uno apps.


## Character Spacing on Uppercase Text

Uppercase and small-caps text needs additional letter-spacing for readability. In XAML, CharacterSpacing is in 1/1000th of an em.

**Incorrect (uppercase text feels cramped):**

```xml
<TextBlock Text="SECTION HEADER"
           FontWeight="SemiBold" />
```

**Correct (added character spacing for breathing room):**

```xml
<TextBlock Text="SECTION HEADER"
           FontWeight="SemiBold"
           CharacterSpacing="50" />
```

---


## Font Fallback Chains

On Uno Platform, fonts may not be available on all targets. Always specify a fallback chain so text renders correctly on every platform.

**Incorrect (single font with no fallback — missing on Android/WASM):**

```xml
<TextBlock FontFamily="Segoe UI" Text="Hello" />
```

**Correct (embedded font with platform fallback):**

```xml
<TextBlock FontFamily="/Assets/Fonts/Inter-Regular.ttf#Inter, Segoe UI, Roboto, San Francisco, Helvetica"
           Text="Hello" />
```

---


## Disable Color Fonts When Inappropriate

IsColorFontEnabled defaults to true, which renders color emoji and color glyphs. In monochrome icon contexts or data-dense UIs, disable it to keep a consistent visual weight.

**Incorrect (color emoji in a monochrome toolbar label):**

```xml
<TextBlock Text="&#x2764; Favorites"
           Foreground="{ThemeResource OnSurfaceBrush}" />
```

**Correct (monochrome glyph matches surrounding icons):**

```xml
<TextBlock Text="&#x2764; Favorites"
           Foreground="{ThemeResource OnSurfaceBrush}"
           IsColorFontEnabled="False" />
```

---


## Explicit Line Height for Consistent Rhythm

Default line height varies across platforms. Set LineHeight explicitly on body text to ensure vertical rhythm is consistent across Windows, Android, iOS, and WASM.

**Incorrect (platform-dependent line height — layout shifts between targets):**

```xml
<TextBlock Text="{x:Bind Body}"
           FontSize="14" />
```

**Correct (explicit line height — consistent on all platforms):**

```xml
<TextBlock Text="{x:Bind Body}"
           FontSize="14"
           LineHeight="20"
           LineStackingStrategy="BlockLineHeight" />
```

---


## Avoid Faux Bold and Italic

Never rely on the platform to synthesize bold or italic from a regular font face. Faux bold thickens strokes uniformly (looks bloated) and faux italic just skews glyphs (looks wrong). Load the actual font weight/style file.

**Incorrect (requesting SemiBold when only Regular is loaded — platform synthesizes):**

```xml
<FontFamily x:Key="AppFont">/Assets/Fonts/Inter-Regular.ttf#Inter</FontFamily>

<TextBlock Text="Important" FontFamily="{StaticResource AppFont}"
           FontWeight="SemiBold" />
```

**Correct (load the actual SemiBold face):**

```xml
<FontFamily x:Key="AppFont">/Assets/Fonts/Inter-SemiBold.ttf#Inter</FontFamily>

<TextBlock Text="Important" FontFamily="{StaticResource AppFont}"
           FontWeight="SemiBold" />
```

---


## Oldstyle Numbers for Prose

Oldstyle (lowercase) numbers have ascenders and descenders that blend naturally into body text. Use them in paragraphs, not in tables.

**Incorrect (lining numbers look mechanical in prose):**

```xml
<TextBlock Text="Founded in 1984 with 27 employees"
           Typography.NumeralStyle="Lining" />
```

**Correct (oldstyle numbers blend into text):**

```xml
<TextBlock Text="Founded in 1984 with 27 employees"
           Typography.NumeralStyle="OldStyle" />
```

---


## Slashed Zero for Code-Adjacent UI

In code-adjacent contexts (IDs, serial numbers, hex values), a slashed zero prevents confusion with the letter O.

**Incorrect (zero and O are indistinguishable):**

```xml
<TextBlock Text="Order #O01230"
           FontFamily="Consolas" />
```

**Correct (slashed zero disambiguates):**

```xml
<TextBlock Text="Order #O01230"
           FontFamily="Consolas"
           Typography.SlashedZero="True" />
```

---


## Tabular Numbers for Data Display

Use Typography.NumeralAlignment="Tabular" for any numeric data that should align in columns (tables, dashboards, pricing). Tabular figures are monospaced so digits stack vertically.

**Incorrect (proportional numbers misalign in columns):**

```xml
<TextBlock Text="{x:Bind Price}" />
```

**Correct (tabular numbers align):**

```xml
<TextBlock Text="{x:Bind Price}"
           Typography.NumeralAlignment="Tabular" />
```

---


## Text Trimming for Overflow

Always set TextTrimming on text that can overflow its container. CharacterEllipsis is safest; WordEllipsis looks cleaner but can hide single long words entirely.

**Incorrect (text overflows or clips silently):**

```xml
<TextBlock Text="{x:Bind Title}"
           MaxLines="1" />
```

**Correct (ellipsis signals truncation):**

```xml
<TextBlock Text="{x:Bind Title}"
           MaxLines="1"
           TextTrimming="CharacterEllipsis" />
```

---


## Text Wrapping for Multi-Line Content

TextBlock defaults to NoWrap. Body text, descriptions, and any multi-line content must explicitly set TextWrapping="WrapWholeWords" to avoid horizontal overflow.

**Incorrect (text runs off-screen on narrow layouts):**

```xml
<TextBlock Text="{x:Bind Description}" />
```

**Correct (wraps at word boundaries):**

```xml
<TextBlock Text="{x:Bind Description}"
           TextWrapping="WrapWholeWords" />
```

---


## Text Alignment Matches Reading Direction

Use TextAlignment="DetectFromContent" or leave at default (Start) for body text. Never hard-code Left for localizable content — it breaks RTL layouts.

**Incorrect (hard-coded Left breaks Arabic/Hebrew):**

```xml
<TextBlock Text="{x:Bind Description}"
           TextAlignment="Left" />
```

**Correct (Start follows FlowDirection automatically):**

```xml
<TextBlock Text="{x:Bind Description}"
           TextAlignment="Start" />
```

---


## Use Variable Fonts for Continuous Weight

Variable fonts contain all weights in a single file, reducing bundle size and enabling smooth weight transitions. Use a single .ttf and set any FontWeight value from 100-900.

**Incorrect (loading 4 separate static font files):**

```xml
<FontFamily x:Key="AppFontLight">/Assets/Fonts/Inter-Light.ttf#Inter</FontFamily>
<FontFamily x:Key="AppFontRegular">/Assets/Fonts/Inter-Regular.ttf#Inter</FontFamily>
<FontFamily x:Key="AppFontMedium">/Assets/Fonts/Inter-Medium.ttf#Inter</FontFamily>
<FontFamily x:Key="AppFontBold">/Assets/Fonts/Inter-Bold.ttf#Inter</FontFamily>
```

**Correct (single variable font, any weight):**

```xml
<FontFamily x:Key="AppFont">/Assets/Fonts/Inter-Variable.ttf#Inter</FontFamily>

<TextBlock FontFamily="{StaticResource AppFont}" FontWeight="350" />
<TextBlock FontFamily="{StaticResource AppFont}" FontWeight="600" />
```

---

## 2. Transitions & Exit Animations

**Impact:** HIGH
**Description:** ThemeTransition, Storyboard exit patterns, and ConnectedAnimationService for cross-page transitions.


## AddDeleteThemeTransition for Collection Changes

When items are added to or removed from a collection, use AddDeleteThemeTransition so the list animates smoothly instead of snapping.

**Incorrect (list items appear/disappear instantly):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <TextBlock Text="{x:Bind Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**Correct (items animate in and out):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemContainerTransitions>
        <AddDeleteThemeTransition />
    </ListView.ItemContainerTransitions>
    <ListView.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <TextBlock Text="{x:Bind Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

---


## ConnectedAnimation for Cross-Page Transitions

ConnectedAnimationService creates fluid transitions between pages by animating a shared element from source to destination. This provides visual continuity during navigation.

**Incorrect (no visual continuity — hard cut between pages):**

```csharp
// SourcePage.xaml.cs
private void OnItemClick(object sender, ItemClickEventArgs e)
{
    Frame.Navigate(typeof(DetailPage), e.ClickedItem);
}
```

**Correct (shared element animates across pages):**

```csharp
// SourcePage.xaml.cs
private void OnItemClick(object sender, ItemClickEventArgs e)
{
    var service = ConnectedAnimationService.GetForCurrentView();
    service.PrepareToAnimate("itemImage", ClickedImage);
    Frame.Navigate(typeof(DetailPage), e.ClickedItem);
}

// DetailPage.xaml.cs
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    var animation = ConnectedAnimationService.GetForCurrentView()
        .GetAnimation("itemImage");
    animation?.TryStart(HeroImage);
}
```

---


## Configure ConnectedAnimation Easing

ConnectedAnimation defaults to a system curve, but for gesture-driven transitions use a spring configuration. Match the animation personality to the interaction type.

**Incorrect (default easing for a drag-to-dismiss — feels stiff):**

```csharp
var animation = ConnectedAnimationService.GetForCurrentView()
    .GetAnimation("card");
animation?.TryStart(TargetElement);
```

**Correct (gravity spring for natural gesture follow-through):**

```csharp
var animation = ConnectedAnimationService.GetForCurrentView()
    .GetAnimation("card");
if (animation is not null)
{
    animation.Configuration = new DirectConnectedAnimationConfiguration();
    animation.TryStart(TargetElement);
}
```

---


## Disable Interactions on Exiting Elements

Elements playing an exit animation are still in the visual tree and can receive pointer events. Disable hit-testing to prevent ghost clicks during exit.

**Incorrect (fading element still receives taps):**

```csharp
private void RemoveCard(UIElement card)
{
    var sb = CreateFadeOutStoryboard(card);
    sb.Completed += (_, _) => Panel.Children.Remove(card);
    sb.Begin();
}
```

**Correct (disable hit-testing immediately):**

```csharp
private void RemoveCard(UIElement card)
{
    card.IsHitTestVisible = false;
    var sb = CreateFadeOutStoryboard(card);
    sb.Completed += (_, _) => Panel.Children.Remove(card);
    sb.Begin();
}
```

---


## Exit Mirrors Entrance for Symmetry

If an element enters by sliding up and fading in, it should exit by sliding down and fading out. Asymmetric enter/exit feels disorienting.

**Incorrect (enters from bottom, exits with scale — inconsistent):**

```xml
<!-- Entrance -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)"
                 From="40" To="0" Duration="0:0:0.25" />

<!-- Exit -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleX)"
                 From="1" To="0" Duration="0:0:0.25" />
```

**Correct (exit mirrors entrance — slide back down):**

```xml
<!-- Entrance -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)"
                 From="40" To="0" Duration="0:0:0.25" />

<!-- Exit -->
<DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)"
                 From="0" To="40" Duration="0:0:0.2" />
```

---


## Page Transition Consistency

Set Frame.ContentTransitions once rather than per-page. Mixed transitions across pages feel chaotic.

**Incorrect (different transitions per page — no consistency):**

```csharp
// PageA
Frame.Navigate(typeof(PageB));
// PageB sets its own transition in OnNavigatedTo
this.Transitions = new TransitionCollection { new EdgeUIThemeTransition() };
```

**Correct (consistent transition on Frame):**

```xml
<Frame x:Name="RootFrame">
    <Frame.ContentTransitions>
        <NavigationThemeTransition>
            <NavigationThemeTransition.DefaultNavigationTransitionInfo>
                <SlideNavigationTransitionInfo Effect="FromRight" />
            </NavigationThemeTransition.DefaultNavigationTransitionInfo>
        </NavigationThemeTransition>
    </Frame.ContentTransitions>
</Frame>
```

---


## Popup and Flyout Exit Animations

Popups and Flyouts should have explicit exit transitions. The default is fast enough for most cases — avoid adding slow custom exits that block interaction.

**Incorrect (custom long exit delays returning to content):**

```xml
<Flyout>
    <Flyout.FlyoutPresenterStyle>
        <Style TargetType="FlyoutPresenter">
            <Setter Property="Transitions">
                <Setter.Value>
                    <TransitionCollection>
                        <PopupThemeTransition />
                    </TransitionCollection>
                </Setter.Value>
            </Setter>
        </Style>
    </Flyout.FlyoutPresenterStyle>
</Flyout>
```

**Correct (rely on built-in popup transitions, keep it snappy):**

```xml
<Flyout LightDismissOverlayMode="On">
    <!-- Built-in PopupThemeTransition is already applied by default.
         Only override if you need to suppress it. -->
    <StackPanel Padding="16" Spacing="8">
        <TextBlock Text="Confirm action?" />
        <Button Content="OK" />
    </StackPanel>
</Flyout>
```

---


## RepositionThemeTransition for Layout Shifts

When sibling elements shift position (e.g., after an item is removed or inserted), RepositionThemeTransition animates remaining items to their new positions instead of snapping.

**Incorrect (remaining items jump after removal):**

```xml
<ItemsRepeater ItemsSource="{x:Bind Items}">
    <ItemsRepeater.ItemTemplate>
        <DataTemplate x:DataType="local:Item">
            <Border Padding="8">
                <TextBlock Text="{x:Bind Name}" />
            </Border>
        </DataTemplate>
    </ItemsRepeater.ItemTemplate>
</ItemsRepeater>
```

**Correct (siblings slide into new positions):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemContainerTransitions>
        <AddDeleteThemeTransition />
        <RepositionThemeTransition />
    </ListView.ItemContainerTransitions>
</ListView>
```

---


## Handle Storyboard.Completed Before Removing Elements

When using a custom exit animation via Storyboard, never remove the element from the visual tree until the Storyboard fires Completed. Removing early aborts the animation.

**Incorrect (removes element immediately — animation never plays):**

```csharp
private void RemoveItem(UIElement element)
{
    var fadeOut = new DoubleAnimation
    {
        To = 0, Duration = TimeSpan.FromMilliseconds(200)
    };
    Storyboard.SetTarget(fadeOut, element);
    Storyboard.SetTargetProperty(fadeOut, "Opacity");

    var sb = new Storyboard { Children = { fadeOut } };
    sb.Begin();
    Panel.Children.Remove(element); // Too early!
}
```

**Correct (waits for Completed then removes):**

```csharp
private void RemoveItem(UIElement element)
{
    var fadeOut = new DoubleAnimation
    {
        To = 0, Duration = TimeSpan.FromMilliseconds(200)
    };
    Storyboard.SetTarget(fadeOut, element);
    Storyboard.SetTargetProperty(fadeOut, "Opacity");

    var sb = new Storyboard { Children = { fadeOut } };
    sb.Completed += (_, _) => Panel.Children.Remove(element);
    sb.Begin();
}
```

---


## Use ThemeTransition for Standard Enter/Exit

WinUI provides built-in ThemeTransitions that handle entrance and exit automatically. Use them instead of writing custom Storyboards for standard patterns.

**Incorrect (no transitions — elements pop in/out abruptly):**

```xml
<StackPanel>
    <TextBlock Text="Welcome" />
</StackPanel>
```

**Correct (theme transitions for smooth entrance):**

```xml
<StackPanel>
    <StackPanel.ChildrenTransitions>
        <EntranceThemeTransition />
    </StackPanel.ChildrenTransitions>
    <TextBlock Text="Welcome" />
</StackPanel>
```

---

## 3. Visual States & Overlays

**Impact:** HIGH
**Description:** VisualStateManager patterns, template parts, and overlay techniques replacing CSS pseudo-elements in XAML.


## Use AdaptiveTrigger for Responsive States

AdaptiveTrigger replaces CSS media queries in XAML. Use it to switch visual states based on window width, replacing code-behind SizeChanged handlers.

**Incorrect (manual breakpoint logic in code-behind):**

```csharp
private void OnSizeChanged(object sender, SizeChangedEventArgs e)
{
    if (e.NewSize.Width < 640)
    {
        SidePanel.Visibility = Visibility.Collapsed;
        BottomNav.Visibility = Visibility.Visible;
    }
    else
    {
        SidePanel.Visibility = Visibility.Visible;
        BottomNav.Visibility = Visibility.Collapsed;
    }
}
```

**Correct (declarative adaptive triggers):**

```xml
<VisualStateGroup x:Name="LayoutStates">
    <VisualState x:Name="Wide">
        <VisualState.StateTriggers>
            <AdaptiveTrigger MinWindowWidth="640" />
        </VisualState.StateTriggers>
        <VisualState.Setters>
            <Setter Target="SidePanel.Visibility" Value="Visible" />
            <Setter Target="BottomNav.Visibility" Value="Collapsed" />
        </VisualState.Setters>
    </VisualState>
    <VisualState x:Name="Narrow">
        <VisualState.StateTriggers>
            <AdaptiveTrigger MinWindowWidth="0" />
        </VisualState.StateTriggers>
        <VisualState.Setters>
            <Setter Target="SidePanel.Visibility" Value="Collapsed" />
            <Setter Target="BottomNav.Visibility" Value="Visible" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```

---


## Use GoToState Not Direct Property Changes

When you must trigger state changes from code, use VisualStateManager.GoToState rather than setting properties directly. GoToState respects transitions and keeps state consistent.

**Incorrect (direct property set — bypasses VSM transitions):**

```csharp
private void OnError()
{
    ErrorBorder.BorderBrush = new SolidColorBrush(Colors.Red);
    ErrorIcon.Visibility = Visibility.Visible;
}
```

**Correct (GoToState triggers defined visual state with transitions):**

```csharp
private void OnError()
{
    VisualStateManager.GoToState(this, "ErrorState", useTransitions: true);
}
```

---


## Expand Hit Targets with Transparent Fill

Small interactive elements need expanded hit areas. In CSS you'd use pseudo-element padding. In XAML, use a transparent Border or set Padding on the clickable container. Minimum 32x32px for touch (44x44 recommended).

**Incorrect (16px icon button — impossible to tap on mobile):**

```xml
<Button Padding="0" MinWidth="0" MinHeight="0">
    <FontIcon Glyph="&#xE72D;" FontSize="16" />
</Button>
```

**Correct (expanded touch target with padding):**

```xml
<Button Padding="12" MinWidth="44" MinHeight="44">
    <FontIcon Glyph="&#xE72D;" FontSize="16" />
</Button>
```

---


## Lightweight Styling Over Full Template Override

Override specific theme resources to restyle controls instead of copying entire ControlTemplates. Lightweight styling survives framework updates and reduces XAML bloat.

**Incorrect (copying entire Button template just to change background):**

```xml
<Style x:Key="AccentButton" TargetType="Button">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <!-- 80+ lines of copied template just to change one brush -->
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

**Correct (override just the resource key):**

```xml
<Button Content="Save">
    <Button.Resources>
        <SolidColorBrush x:Key="ButtonBackground"
                         Color="{ThemeResource SystemAccentColor}" />
        <SolidColorBrush x:Key="ButtonBackgroundPointerOver"
                         Color="{ThemeResource SystemAccentColorLight1}" />
        <SolidColorBrush x:Key="ButtonBackgroundPressed"
                         Color="{ThemeResource SystemAccentColorDark1}" />
    </Button.Resources>
</Button>
```

---


## Use Overlay Shapes Instead of Extra Elements

In CSS, ::before/::after create decorative overlays without DOM nodes. In XAML, use a Border or Rectangle in the same Grid cell to achieve the same effect without adding a wrapper element.

**Incorrect (extra wrapping StackPanel just for a highlight overlay):**

```xml
<StackPanel>
    <Border Background="Gold" Opacity="0.3" Height="40" />
    <TextBlock Text="Featured" Margin="0,-40,0,0" />
</StackPanel>
```

**Correct (overlay in same Grid cell — clean and layered):**

```xml
<Grid>
    <Border Background="{ThemeResource SystemAccentColorLight2}"
            Opacity="0.15"
            CornerRadius="4" />
    <TextBlock Text="Featured"
               Padding="8,4"
               VerticalAlignment="Center" />
</Grid>
```

---


## Define All Interactive States

Custom interactive controls must define Normal, PointerOver, Pressed, Disabled, and Focused states. Missing states create dead spots where the control feels unresponsive.

**Incorrect (only Normal and PointerOver — Pressed and Disabled feel broken):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```

**Correct (complete state set):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
    <VisualState x:Name="Pressed">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorTertiaryBrush}" />
            <Setter Target="Root.RenderTransform">
                <Setter.Value>
                    <ScaleTransform ScaleX="0.98" ScaleY="0.98" />
                </Setter.Value>
            </Setter>
        </VisualState.Setters>
    </VisualState>
    <VisualState x:Name="Disabled">
        <VisualState.Setters>
            <Setter Target="Root.Opacity" Value="0.4" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```

---


## Use TemplatePart for Reusable Control Decorations

When building custom controls that need decorative elements (indicators, badges, selection marks), define them as named parts in the ControlTemplate rather than adding ad-hoc elements in each instance.

**Incorrect (decoration duplicated per instance):**

```xml
<Grid>
    <local:StatusCard />
    <Ellipse Width="8" Height="8" Fill="Red"
             HorizontalAlignment="Right" VerticalAlignment="Top" />
</Grid>
```

**Correct (decoration is a template part inside the control):**

```csharp
[TemplatePart(Name = "PART_Badge", Type = typeof(Ellipse))]
public sealed class StatusCard : Control { ... }
```

```xml
<Style TargetType="local:StatusCard">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="local:StatusCard">
                <Grid>
                    <ContentPresenter />
                    <Ellipse x:Name="PART_Badge"
                             Width="8" Height="8" Fill="Red"
                             HorizontalAlignment="Right"
                             VerticalAlignment="Top"
                             Visibility="Collapsed" />
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

---


## Use VisualStateManager for State-Driven Styling

Use VisualStateManager instead of code-behind property changes for interactive states. VSM centralizes state definitions, enables animation, and works with adaptive triggers.

**Incorrect (toggling properties in code-behind — scattered, not animatable):**

```csharp
private void OnPointerEntered(object sender, PointerRoutedEventArgs e)
{
    MyBorder.Background = new SolidColorBrush(Colors.LightBlue);
    MyBorder.BorderThickness = new Thickness(2);
}
```

**Correct (VisualStateManager in XAML — declarative and animatable):**

```xml
<Border x:Name="MyBorder" PointerEntered="OnPointerEntered">
    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup x:Name="HoverStates">
            <VisualState x:Name="Normal" />
            <VisualState x:Name="PointerOver">
                <VisualState.Setters>
                    <Setter Target="MyBorder.Background"
                            Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
                    <Setter Target="MyBorder.BorderThickness" Value="2" />
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>
</Border>
```

---


## Animate Between Visual States with Transitions

VisualState.Setters snap instantly. For smooth state changes, add VisualTransitions with duration between states.

**Incorrect (background snaps on hover — jarring):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```

**Correct (smooth transition between states):**

```xml
<VisualStateGroup x:Name="CommonStates">
    <VisualStateGroup.Transitions>
        <VisualTransition GeneratedDuration="0:0:0.15" />
    </VisualStateGroup.Transitions>
    <VisualState x:Name="Normal" />
    <VisualState x:Name="PointerOver">
        <VisualState.Setters>
            <Setter Target="Root.Background"
                    Value="{ThemeResource SubtleFillColorSecondaryBrush}" />
        </VisualState.Setters>
    </VisualState>
</VisualStateGroup>
```

---

## 4. Layout Animation

**Impact:** MEDIUM
**Description:** Composition implicit animations, SizeChanged-driven animation, and the measure/animate separation pattern.


## Don't Animate Layout on Virtualized Panels

Virtualized panels (ListView, GridView, ItemsRepeater with virtualizing layout) recycle containers. Implicit layout animations on recycled containers cause ghost animations and visual glitches.

**Incorrect (implicit offset animation on ListView items — recycled items ghost-slide):**

```csharp
private void OnContainerContentChanging(ListViewBase sender,
    ContainerContentChangingEventArgs args)
{
    var visual = ElementCompositionPreview.GetElementVisual(args.ItemContainer);
    var group = visual.Compositor.CreateImplicitAnimationCollection();
    group["Offset"] = CreateSpringAnimation(visual.Compositor);
    visual.ImplicitAnimations = group; // Bad on recycled containers
}
```

**Correct (use built-in ItemContainerTransitions instead):**

```xml
<ListView ItemsSource="{x:Bind Items}">
    <ListView.ItemContainerTransitions>
        <AddDeleteThemeTransition />
        <RepositionThemeTransition />
    </ListView.ItemContainerTransitions>
</ListView>
```

---


## Clip Content During Animated Resize

When animating container size, content can overflow during the transition. Set Clip on the container to prevent visual bleed.

**Incorrect (content overflows during height animation):**

```xml
<Border x:Name="ExpandablePanel" Height="0">
    <StackPanel>
        <TextBlock Text="This leaks out during animation" />
    </StackPanel>
</Border>
```

**Correct (clip constrains content during resize):**

```xml
<Border x:Name="ExpandablePanel" Height="0">
    <Border.Clip>
        <RectangleGeometry Rect="0,0,9999,0"
                           x:Name="PanelClip" />
    </Border.Clip>
    <StackPanel>
        <TextBlock Text="Properly clipped during animation" />
    </StackPanel>
</Border>
```

```csharp
// Animate both Height and Clip rect together
private void Expand(double targetHeight)
{
    PanelClip.Rect = new Rect(0, 0, 9999, targetHeight);
    // Animate ExpandablePanel.Height to targetHeight
}
```

---


## Composition Animations Over Storyboard for Layout

Composition animations run on the compositor thread, independent of the UI thread. For layout-triggered animations (offset, scale, opacity), prefer composition over XAML Storyboards which run on the UI thread and can cause jank.

**Incorrect (XAML Storyboard for offset — runs on UI thread):**

```xml
<Storyboard x:Name="SlideIn">
    <DoubleAnimation Storyboard.TargetName="MyPanel"
                     Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.X)"
                     From="200" To="0" Duration="0:0:0.3" />
</Storyboard>
```

**Correct (composition animation — runs off UI thread):**

```csharp
var visual = ElementCompositionPreview.GetElementVisual(MyPanel);
var animation = visual.Compositor.CreateScalarKeyFrameAnimation();
animation.InsertKeyFrame(0f, 200f);
animation.InsertKeyFrame(1f, 0f, visual.Compositor.CreateCubicBezierEasingFunction(
    new Vector2(0, 0), new Vector2(0.2f, 1f)));
animation.Duration = TimeSpan.FromMilliseconds(300);
visual.StartAnimation("Translation.X", animation);
```

---


## Use Implicit Animations for Layout Changes

Composition implicit animations automatically animate property changes on a Visual. Use ElementCompositionPreview to attach them so elements glide to new positions instead of snapping.

**Incorrect (elements jump to new positions after layout change):**

```csharp
// No animation — items snap when reordered
myElement.Margin = new Thickness(100, 0, 0, 0);
```

**Correct (implicit animation on Offset — smooth repositioning):**

```csharp
var visual = ElementCompositionPreview.GetElementVisual(myElement);
var compositor = visual.Compositor;

var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(250);

var group = compositor.CreateImplicitAnimationCollection();
group["Offset"] = animation;
visual.ImplicitAnimations = group;
```

---


## Separate Measurement from Animation Target

Never measure and animate the same element — it creates a feedback loop. Use one element for natural size measurement and a parent for the animated bounds.

**Incorrect (animating Height on the element being measured):**

```csharp
private void OnContentSizeChanged(object sender, SizeChangedEventArgs e)
{
    // Animating Height triggers another SizeChanged — infinite loop
    AnimateHeight(ContentPanel, e.NewSize.Height);
}
```

**Correct (inner element measures, outer element animates):**

```xml
<Border x:Name="AnimatedWrapper">
    <StackPanel x:Name="MeasuredContent"
                SizeChanged="OnContentSizeChanged">
        <!-- Dynamic content here -->
    </StackPanel>
</Border>
```

```csharp
private void OnContentSizeChanged(object sender, SizeChangedEventArgs e)
{
    // Animate the wrapper, not the measured content
    AnimateHeight(AnimatedWrapper, e.NewSize.Height);
}
```

---


## Use Spring NaturalMotion for Organic Layout

SpringVector3NaturalMotionAnimation creates organic, interruptible motion. Use it for layout animations that should feel physical and responsive.

**Incorrect (linear keyframe animation — feels mechanical):**

```csharp
var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(300);
```

**Correct (spring motion — organic and interruptible):**

```csharp
var spring = compositor.CreateSpringVector3Animation();
spring.FinalValue = new Vector3(0);
spring.DampingRatio = 0.7f;
spring.Period = TimeSpan.FromMilliseconds(80);

var group = compositor.CreateImplicitAnimationCollection();
group["Offset"] = spring;
visual.ImplicitAnimations = group;
```

---


## Add Slight Delay for Catching-Up Feel

A small delay (30-60ms) before a layout animation starts creates a natural "catching up" feel, as if the element reacts to the change rather than predicting it.

**Incorrect (animation starts simultaneously with trigger — feels robotic):**

```csharp
var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(250);
animation.DelayTime = TimeSpan.Zero;
```

**Correct (small delay for organic feel):**

```csharp
var animation = compositor.CreateVector3KeyFrameAnimation();
animation.InsertExpressionKeyFrame(1f, "this.FinalValue");
animation.Duration = TimeSpan.FromMilliseconds(250);
animation.DelayTime = TimeSpan.FromMilliseconds(40);
animation.DelayBehavior = AnimationDelayBehavior.SetInitialValueBeforeDelay;
```

---

## 5. Data Prefetching

**Impact:** MEDIUM
**Description:** Loading data before user navigates by analyzing pointer trajectory, focus patterns, and incremental loading.


## Prefetch Data on Focus Before Selection

When a user navigates a list with keyboard, the focused item will likely be selected next. Start loading data on GotFocus rather than waiting for SelectionChanged.

**Incorrect (loads data only after selection — keyboard users wait):**

```csharp
private async void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (e.AddedItems.FirstOrDefault() is Category category)
    {
        DetailPanel.Content = await LoadCategoryDetails(category.Id);
    }
}
```

**Correct (prefetch on focus, confirm on selection):**

```csharp
private Task<CategoryDetail>? _prefetchTask;

private void OnItemGotFocus(object sender, RoutedEventArgs e)
{
    if (sender is FrameworkElement { DataContext: Category category })
    {
        _prefetchTask = LoadCategoryDetails(category.Id);
    }
}

private async void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (_prefetchTask is not null)
    {
        DetailPanel.Content = await _prefetchTask;
    }
}
```

---


## ISupportIncrementalLoading for Anticipatory Paging

Implement ISupportIncrementalLoading on your collection so ListView automatically loads the next page before the user scrolls to the end, rather than showing a "load more" button.

**Incorrect (manual load-more button — interrupts scroll flow):**

```xml
<StackPanel>
    <ListView ItemsSource="{x:Bind Items}" />
    <Button Content="Load More" Click="OnLoadMore" />
</StackPanel>
```

**Correct (incremental loading — seamless infinite scroll):**

```csharp
public class IncrementalItemSource : ObservableCollection<Item>,
    ISupportIncrementalLoading
{
    public bool HasMoreItems => _hasMore;

    public IAsyncOperation<LoadMoreItemsResult> LoadMoreItemsAsync(uint count)
    {
        return AsyncInfo.Run(async ct =>
        {
            var items = await _service.GetPageAsync(_page++, (int)count, ct);
            foreach (var item in items) Add(item);
            _hasMore = items.Count == count;
            return new LoadMoreItemsResult { Count = (uint)items.Count };
        });
    }
}
```

```xml
<ListView ItemsSource="{x:Bind IncrementalItems}"
          IncrementalLoadingTrigger="Edge"
          DataFetchSize="2" />
```

---


## Prefetch by Intent Not Viewport

Don't prefetch every visible navigation target. Prefetch based on user intent signals (trajectory, focus, frequency) to avoid wasted bandwidth and API calls.

**Incorrect (prefetches all visible items on load — wasteful):**

```csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    foreach (var item in NavigationItems)
    {
        _ = PrefetchService.LoadAsync(item.Route); // 20 API calls at once
    }
}
```

**Correct (prefetch only the most likely target):**

```csharp
private void OnNavigationItemGotFocus(object sender, RoutedEventArgs e)
{
    if (sender is FrameworkElement { DataContext: NavigationItem item })
    {
        _ = PrefetchService.LoadIfNotCachedAsync(item.Route);
    }
}
```

---


## Pointer Trajectory Prediction Over PointerEntered

PointerEntered fires only when the cursor reaches the element. Track PointerMoved on a parent and predict which child the cursor is heading toward to start prefetching 100-200ms earlier.

**Incorrect (prefetches on hover — too late):**

```csharp
private void OnItemPointerEntered(object sender, PointerRoutedEventArgs e)
{
    if (sender is FrameworkElement { DataContext: NavigationItem item })
    {
        _ = PrefetchService.LoadAsync(item.Route);
    }
}
```

**Correct (trajectory prediction starts prefetch earlier):**

```csharp
private Point _lastPosition;

private void OnPanelPointerMoved(object sender, PointerRoutedEventArgs e)
{
    var current = e.GetCurrentPoint(NavigationPanel).Position;
    var velocity = new Point(current.X - _lastPosition.X, current.Y - _lastPosition.Y);
    _lastPosition = current;

    var predicted = new Point(current.X + velocity.X * 5, current.Y + velocity.Y * 5);
    var target = VisualTreeHelper.FindElementsInHostCoordinates(predicted, NavigationPanel)
        .OfType<FrameworkElement>()
        .FirstOrDefault(e => e.DataContext is NavigationItem);

    if (target?.DataContext is NavigationItem item)
    {
        _ = PrefetchService.LoadAsync(item.Route);
    }
}
```

---


## Touch Fallback for Pointer Prediction

Pointer trajectory prediction requires a cursor. On touch devices, fall back to focus-based or tap-hint-based prefetching since there's no hover state.

**Incorrect (trajectory prediction fails silently on touch):**

```csharp
private void OnPointerMoved(object sender, PointerRoutedEventArgs e)
{
    // This never fires on tap-only devices
    PredictAndPrefetch(e.GetCurrentPoint(this).Position);
}
```

**Correct (check input type and use appropriate strategy):**

```csharp
private void OnPointerMoved(object sender, PointerRoutedEventArgs e)
{
    if (e.Pointer.PointerDeviceType == PointerDeviceType.Mouse)
    {
        PredictAndPrefetch(e.GetCurrentPoint(this).Position);
    }
    // Touch users get focus-based prefetch via GotFocus handlers
}
```

---

## 6. Animated Icons

**Impact:** MEDIUM
**Description:** AnimatedIcon, Lottie via AnimatedVisualPlayer, and animated path transitions.


## Use AnimatedIcon for State Transitions

AnimatedIcon integrates with control states (PointerOver, Pressed) to play Lottie segments automatically. Use it instead of swapping between static icons on state change.

**Incorrect (swapping icons on state change — no transition):**

```xml
<Button>
    <FontIcon x:Name="PlayIcon" Glyph="&#xE768;" />
</Button>
```

```csharp
private void OnToggle()
{
    PlayIcon.Glyph = _isPlaying ? "\xE768" : "\xE769"; // Hard swap
}
```

**Correct (AnimatedIcon morphs between states):**

```xml
<Button>
    <AnimatedIcon x:Name="PlayPauseIcon">
        <animatedvisuals:AnimatedPlayPauseIcon />
        <AnimatedIcon.FallbackIconSource>
            <FontIconSource Glyph="&#xE768;" />
        </AnimatedIcon.FallbackIconSource>
    </AnimatedIcon>
</Button>
```

---


## Mark Decorative Icons as Not Accessible

Decorative icons (next to a label) should be hidden from screen readers with AccessibilityView="Raw". Only set AutomationProperties.Name on icons that are the sole indicator of meaning.

**Incorrect (icon announces redundantly alongside label):**

```xml
<StackPanel Orientation="Horizontal">
    <FontIcon Glyph="&#xE74D;"
              AutomationProperties.Name="Save" />
    <TextBlock Text="Save" />
</StackPanel>
```

**Correct (decorative icon hidden, label carries meaning):**

```xml
<StackPanel Orientation="Horizontal">
    <FontIcon Glyph="&#xE74D;"
              AutomationProperties.AccessibilityView="Raw" />
    <TextBlock Text="Save" />
</StackPanel>
```

---


## Always Set FallbackIconSource

AnimatedIcon may fail to load its visual on some platforms. Always set FallbackIconSource so the control never renders empty.

**Incorrect (no fallback — blank space if animation fails to load):**

```xml
<AnimatedIcon>
    <animatedvisuals:AnimatedSettingsIcon />
</AnimatedIcon>
```

**Correct (FontIcon fallback ensures something always renders):**

```xml
<AnimatedIcon>
    <animatedvisuals:AnimatedSettingsIcon />
    <AnimatedIcon.FallbackIconSource>
        <FontIconSource Glyph="&#xE713;" />
    </AnimatedIcon.FallbackIconSource>
</AnimatedIcon>
```

---


## AnimatedVisualPlayer for Complex Icon Animation

For custom Lottie animations beyond built-in AnimatedIcon, use AnimatedVisualPlayer with explicit playback control. Set AutoPlay="False" to control when segments play.

**Incorrect (AutoPlay runs animation on load — wasted motion):**

```xml
<AnimatedVisualPlayer x:Name="SuccessAnimation"
                      AutoPlay="True">
    <lottie:LottieVisualSource UriSource="ms-appx:///Assets/success.json" />
</AnimatedVisualPlayer>
```

**Correct (play on demand at the right moment):**

```xml
<AnimatedVisualPlayer x:Name="SuccessAnimation"
                      AutoPlay="False">
    <lottie:LottieVisualSource UriSource="ms-appx:///Assets/success.json" />
</AnimatedVisualPlayer>
```

```csharp
private async Task ShowSuccess()
{
    SuccessAnimation.Visibility = Visibility.Visible;
    await SuccessAnimation.PlayAsync(0, 1, looped: false);
}
```

---


## Respect Reduced Motion for Icon Animations

Check UISettings.AnimationsEnabled before playing icon animations. When disabled, show the final state immediately via the fallback icon.

**Incorrect (always plays animation — ignores user preference):**

```csharp
private async Task AnimateIcon()
{
    await IconPlayer.PlayAsync(0, 1, looped: false);
}
```

**Correct (skip animation when reduced motion is set):**

```csharp
private async Task AnimateIcon()
{
    var settings = new UISettings();
    if (settings.AnimationsEnabled)
    {
        await IconPlayer.PlayAsync(0, 1, looped: false);
    }
    else
    {
        // Jump to final frame
        IconPlayer.SetProgress(1.0);
    }
}
```

---

## 7. XAML Audio

**Impact:** MEDIUM
**Description:** MediaPlayer for UI sound effects and AudioGraph for procedural audio in WinUI/Uno apps.


## Provide Toggle to Disable UI Sounds

Every sound must have a visual equivalent, and users must be able to disable all UI sounds independently of system volume. Store this preference in app settings.

**Incorrect (no way to disable UI sounds):**

```csharp
private void OnSuccess()
{
    _sfxPlayer.Play(); // Always plays, no opt-out
    ShowSuccessVisual();
}
```

**Correct (check user preference, always show visual):**

```csharp
private void OnSuccess()
{
    if (_settings.UISoundsEnabled)
    {
        _sfxPlayer.Position = TimeSpan.Zero;
        _sfxPlayer.Play();
    }
    ShowSuccessVisual(); // Visual feedback always shows
}
```

---


## AudioGraph for Procedural UI Sounds

For dynamic sounds (click tones that vary by context, completion chimes with different pitches), use AudioGraph instead of pre-recorded files. One AudioGraph instance with AudioFrameInputNode gives full control.

**Incorrect (10 pre-recorded files for pitch variations):**

```
Assets/Audio/tone-c4.wav
Assets/Audio/tone-d4.wav
Assets/Audio/tone-e4.wav
... // 10 files for one interaction
```

**Correct (AudioGraph generates pitches dynamically):**

```csharp
private async Task InitAudioGraph()
{
    var result = await AudioGraph.CreateAsync(
        new AudioGraphSettings(AudioRenderCategory.SoundEffects));
    _graph = result.Graph;
    _outputNode = await _graph.CreateDeviceOutputNodeAsync();
    _inputNode = _graph.CreateFrameInputNode();
    _inputNode.AddOutgoingConnection(_outputNode.DeviceOutputNode);
    _graph.Start();
}

private void PlayTone(float frequency, double durationMs)
{
    // Generate sine wave AudioFrame at frequency
    var frame = GenerateSineFrame(frequency, durationMs, _graph.EncodingProperties);
    _inputNode.AddFrame(frame);
}
```

---


## Reuse a Single MediaPlayer Instance

Creating a new MediaPlayer per sound causes allocation overhead and audible delay. Create one instance and swap the source.

**Incorrect (new MediaPlayer per click — GC pressure and latency):**

```csharp
private void PlayClickSound()
{
    var player = new MediaPlayer();
    player.Source = MediaSource.CreateFromUri(new Uri("ms-appx:///Assets/click.wav"));
    player.Play();
}
```

**Correct (reuse single player, reset position):**

```csharp
private readonly MediaPlayer _sfxPlayer = new();

private void PlayClickSound()
{
    _sfxPlayer.Source = MediaSource.CreateFromUri(new Uri("ms-appx:///Assets/click.wav"));
    _sfxPlayer.Position = TimeSpan.Zero;
    _sfxPlayer.Play();
}

// Dispose in page cleanup
public void Dispose() => _sfxPlayer.Dispose();
```

---


## Preload Audio to Avoid First-Play Delay

The first play of a MediaSource has loading latency. Preload sounds during page initialization so they play instantly on interaction.

**Incorrect (loads audio on first click — noticeable delay):**

```csharp
private async void OnButtonClick(object sender, RoutedEventArgs e)
{
    var player = new MediaPlayer();
    player.Source = MediaSource.CreateFromUri(new Uri("ms-appx:///Assets/success.wav"));
    player.Play(); // First play is delayed
}
```

**Correct (preload during init, play instantly later):**

```csharp
private readonly MediaPlayer _successPlayer = new();

protected override void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    _successPlayer.Source = MediaSource.CreateFromUri(
        new Uri("ms-appx:///Assets/success.wav"));
    _successPlayer.Volume = 0.3;
}

private void OnSuccess()
{
    _successPlayer.Position = TimeSpan.Zero;
    _successPlayer.Play(); // Instant
}
```

---


## Default Volume Should Be Subtle

UI sound effects should be background confirmation, not foreground noise. Default to 0.3 volume, not 1.0.

**Incorrect (full volume — startles user):**

```csharp
_sfxPlayer.Volume = 1.0;
_sfxPlayer.Play();
```

**Correct (subtle default — confirms without annoying):**

```csharp
_sfxPlayer.Volume = 0.3;
_sfxPlayer.Play();
```

---

## 8. Visual Design XAML

**Impact:** HIGH
**Description:** ThemeShadow, CornerRadius nesting, spacing scales, and brush patterns in XAML.


## Semi-Transparent Borders Adapt to Any Background

Hard-coded border colors break on theme or background changes. Use semi-transparent brushes so borders adapt automatically.

**Incorrect (hard-coded gray — wrong on dark theme):**

```xml
<Border BorderBrush="#CCCCCC" BorderThickness="1">
    <TextBlock Text="Card" />
</Border>
```

**Correct (semi-transparent adapts to theme):**

```xml
<Border BorderBrush="{ThemeResource CardStrokeColorDefaultBrush}"
        BorderThickness="1">
    <TextBlock Text="Card" />
</Border>
```

Or with a custom alpha brush:

```xml
<SolidColorBrush x:Key="SubtleBorder" Color="{ThemeResource SystemBaseHighColor}" Opacity="0.1" />
```

---


## Concentric CornerRadius for Nested Elements

Inner CornerRadius = outer CornerRadius minus padding. This creates concentric curves that look intentional. Matching inner/outer radius looks wrong because the curves don't align.

**Incorrect (same radius on both — inner curve looks too round):**

```xml
<Border CornerRadius="16" Padding="8"
        Background="{ThemeResource CardBackgroundFillColorDefaultBrush}">
    <Border CornerRadius="16"
            Background="{ThemeResource SolidBackgroundFillColorBaseBrush}">
        <TextBlock Text="Content" Padding="12" />
    </Border>
</Border>
```

**Correct (inner radius = 16 - 8 = 8):**

```xml
<Border CornerRadius="16" Padding="8"
        Background="{ThemeResource CardBackgroundFillColorDefaultBrush}">
    <Border CornerRadius="8"
            Background="{ThemeResource SolidBackgroundFillColorBaseBrush}">
        <TextBlock Text="Content" Padding="12" />
    </Border>
</Border>
```

---


## Shadow Matches Elevation in Consistent Scale

Define elevation levels (0, 1, 2, 4, 8, 16, 32) and use consistent Translation.Z values. Arbitrary Z values break the spatial model.

**Incorrect (random Z values with no relationship):**

```xml
<Border Translation="0,0,7">...</Border>   <!-- What level is this? -->
<Border Translation="0,0,23">...</Border>  <!-- Higher than what? -->
<Border Translation="0,0,3">...</Border>   <!-- No pattern -->
```

**Correct (defined elevation levels):**

```xml
<!-- Elevation level 0: flat (no shadow) -->
<Border Translation="0,0,0">...</Border>

<!-- Elevation level 1: subtle (cards, list items) -->
<Border Translation="0,0,4">...</Border>

<!-- Elevation level 2: raised (FAB, app bar) -->
<Border Translation="0,0,8">...</Border>

<!-- Elevation level 3: overlay (dialogs, menus) -->
<Border Translation="0,0,16">...</Border>

<!-- Elevation level 4: top (drawers, navigation) -->
<Border Translation="0,0,32">...</Border>
```

---


## No Pure Black Shadows

Pure black shadows (#000) look harsh and unnatural. Use semi-transparent neutral colors that adapt to the background.

**Incorrect (pure black shadow — too harsh):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Color = Colors.Black;
shadow.BlurRadius = 12;
```

**Correct (semi-transparent neutral — soft and natural):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Color = Color.FromArgb(40, 0, 0, 0); // 15% opacity
shadow.BlurRadius = 12;
```

---


## Use Uno Toolkit ShadowContainer for Layered Shadows

Multiple layered shadows create more realistic depth than a single shadow. Uno Toolkit's ShadowContainer supports multiple shadow definitions in XAML.

**Incorrect (single flat shadow — lacks depth):**

```xml
<Border Translation="0,0,16">
    <Border.Shadow>
        <ThemeShadow />
    </Border.Shadow>
    <TextBlock Text="Card" />
</Border>
```

**Correct (layered shadows for nuanced depth):**

```xml
<utu:ShadowContainer>
    <utu:ShadowContainer.Shadows>
        <utu:ShadowCollection>
            <utu:Shadow BlurRadius="2" OffsetY="1"
                        Color="#20000000" />
            <utu:Shadow BlurRadius="8" OffsetY="4"
                        Color="#14000000" />
            <utu:Shadow BlurRadius="24" OffsetY="8"
                        Color="#0A000000" />
        </utu:ShadowCollection>
    </utu:ShadowContainer.Shadows>
    <Border CornerRadius="8" Padding="16"
            Background="{ThemeResource CardBackgroundFillColorDefaultBrush}">
        <TextBlock Text="Card" />
    </Border>
</utu:ShadowContainer>
```

---


## Consistent Shadow Direction

All shadows in the app should share the same offset direction (single virtual light source). ThemeShadow handles this automatically — avoid mixing it with custom DropShadow that has a different offset.

**Incorrect (custom shadow with different light angle than ThemeShadow):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Offset = new Vector3(-4, -4, 0); // Light from bottom-right
// Meanwhile ThemeShadow assumes light from top
```

**Correct (stick to ThemeShadow or use consistent custom offsets):**

```csharp
var shadow = compositor.CreateDropShadow();
shadow.Offset = new Vector3(0, 2, 0); // Light from top, matching system
shadow.BlurRadius = 8;
shadow.Color = Color.FromArgb(40, 0, 0, 0);
```

---


## Use a Consistent Spacing Scale

Define spacing as resource doubles (4, 8, 12, 16, 24, 32, 48) and reference them throughout. Arbitrary margin/padding values create visual inconsistency.

**Incorrect (arbitrary values — 7px here, 13px there):**

```xml
<StackPanel Margin="7,13,7,5">
    <TextBlock Margin="0,0,0,9" Text="Title" />
    <TextBlock Margin="0,0,0,11" Text="Subtitle" />
</StackPanel>
```

**Correct (consistent scale from resources):**

```xml
<!-- In ResourceDictionary -->
<x:Double x:Key="SpacingXS">4</x:Double>
<x:Double x:Key="SpacingSM">8</x:Double>
<x:Double x:Key="SpacingMD">12</x:Double>
<x:Double x:Key="SpacingLG">16</x:Double>
<x:Double x:Key="SpacingXL">24</x:Double>

<!-- Usage -->
<StackPanel Margin="{StaticResource SpacingLG}">
    <TextBlock Margin="0,0,0,{StaticResource SpacingSM}" Text="Title" />
    <TextBlock Margin="0,0,0,{StaticResource SpacingSM}" Text="Subtitle" />
</StackPanel>
```

---


## Use ThemeShadow for Elevation

ThemeShadow provides system-consistent elevation that adapts to light/dark theme. Use it with Translation.Z to set elevation level rather than hardcoding shadow parameters.

**Incorrect (no elevation — card floats without visual grounding):**

```xml
<Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
        CornerRadius="8" Padding="16">
    <TextBlock Text="Card content" />
</Border>
```

**Correct (ThemeShadow with Z translation):**

```xml
<Border Background="{ThemeResource CardBackgroundFillColorDefaultBrush}"
        CornerRadius="8" Padding="16"
        Translation="0,0,32">
    <Border.Shadow>
        <ThemeShadow />
    </Border.Shadow>
    <TextBlock Text="Card content" />
</Border>
```

---

