---
title: Popup and Flyout Exit Animations
impact: MEDIUM
tags: exit, popup, flyout, dialog
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
