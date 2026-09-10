---
title: Use TemplatePart for Reusable Control Decorations
impact: MEDIUM
tags: state, template-part, control-template, decoration
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
