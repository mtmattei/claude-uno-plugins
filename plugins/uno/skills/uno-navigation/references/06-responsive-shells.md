# Responsive Navigation Shells

How to build adaptive navigation that switches between TabBar on mobile and NavigationView on desktop based on screen width.

---

## Concept

**Impact**: HIGH — Most production apps need different navigation patterns for mobile vs desktop.

On narrow screens (phones), a bottom TabBar is standard. On wider screens (tablets, desktops), a sidebar NavigationView is expected. Both controls navigate to the same content using shared `Region.Name` values.

### Common Breakpoints

| Breakpoint | Width | Pattern |
|---|---|---|
| Narrow (mobile) | < 700px | Bottom TabBar |
| Normal (tablet) | 700 - 999px | NavigationView (collapsed pane) |
| Wide (desktop) | >= 1000px | NavigationView (expanded pane) |

---

## VisualStateManager Approach

**Impact**: CRITICAL — The primary technique for responsive shells. Uses AdaptiveTrigger to show/hide navigation controls at breakpoints.

### Complete Example

```xml
<!-- CORRECT: Responsive shell with VisualStateManager -->
<Grid uen:Region.Attached="True">

    <VisualStateManager.VisualStateGroups>
        <VisualStateGroup>
            <VisualState x:Name="Narrow">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="0" />
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="TabBarNavigation.Visibility" Value="Visible" />
                    <Setter Target="SidebarNavigation.Visibility" Value="Collapsed" />
                    <Setter Target="SidebarNavigation.IsPaneOpen" Value="False" />
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="Normal">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="700" />
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="TabBarNavigation.Visibility" Value="Collapsed" />
                    <Setter Target="SidebarNavigation.Visibility" Value="Visible" />
                    <Setter Target="SidebarNavigation.IsPaneOpen" Value="False" />
                </VisualState.Setters>
            </VisualState>
            <VisualState x:Name="Wide">
                <VisualState.StateTriggers>
                    <AdaptiveTrigger MinWindowWidth="1000" />
                </VisualState.StateTriggers>
                <VisualState.Setters>
                    <Setter Target="TabBarNavigation.Visibility" Value="Collapsed" />
                    <Setter Target="SidebarNavigation.Visibility" Value="Visible" />
                    <Setter Target="SidebarNavigation.IsPaneOpen" Value="True" />
                </VisualState.Setters>
            </VisualState>
        </VisualStateGroup>
    </VisualStateManager.VisualStateGroups>

    <!-- Desktop/Tablet: NavigationView with Frame -->
    <NavigationView x:Name="SidebarNavigation"
                    uen:Region.Attached="True"
                    Visibility="Collapsed">
        <NavigationView.MenuItems>
            <NavigationViewItem Content="Home" uen:Region.Name="Home">
                <NavigationViewItem.Icon>
                    <SymbolIcon Symbol="Home" />
                </NavigationViewItem.Icon>
            </NavigationViewItem>
            <NavigationViewItem Content="Search" uen:Region.Name="Search">
                <NavigationViewItem.Icon>
                    <SymbolIcon Symbol="Find" />
                </NavigationViewItem.Icon>
            </NavigationViewItem>
            <NavigationViewItem Content="Profile" uen:Region.Name="Profile">
                <NavigationViewItem.Icon>
                    <SymbolIcon Symbol="Contact" />
                </NavigationViewItem.Icon>
            </NavigationViewItem>
        </NavigationView.MenuItems>
        <Frame uen:Region.Attached="True" />
    </NavigationView>

    <!-- Mobile: TabBar with Visibility navigator -->
    <Grid x:Name="TabBarNavigation" Visibility="Visible">
        <Grid uen:Region.Attached="True"
              uen:Region.Navigator="Visibility">
            <Grid uen:Region.Name="Home" Visibility="Collapsed">
                <local:HomePage />
            </Grid>
            <Grid uen:Region.Name="Search" Visibility="Collapsed">
                <local:SearchPage />
            </Grid>
            <Grid uen:Region.Name="Profile" Visibility="Collapsed">
                <local:ProfilePage />
            </Grid>
        </Grid>
        <utu:TabBar uen:Region.Attached="True"
                    VerticalAlignment="Bottom"
                    Style="{StaticResource MaterialBottomTabBarStyle}">
            <utu:TabBarItem uen:Region.Name="Home" Content="Home"
                            Style="{StaticResource MaterialBottomTabBarItemStyle}">
                <utu:TabBarItem.Icon>
                    <SymbolIcon Symbol="Home" />
                </utu:TabBarItem.Icon>
            </utu:TabBarItem>
            <utu:TabBarItem uen:Region.Name="Search" Content="Search"
                            Style="{StaticResource MaterialBottomTabBarItemStyle}">
                <utu:TabBarItem.Icon>
                    <SymbolIcon Symbol="Find" />
                </utu:TabBarItem.Icon>
            </utu:TabBarItem>
            <utu:TabBarItem uen:Region.Name="Profile" Content="Profile"
                            Style="{StaticResource MaterialBottomTabBarItemStyle}">
                <utu:TabBarItem.Icon>
                    <SymbolIcon Symbol="Contact" />
                </utu:TabBarItem.Icon>
            </utu:TabBarItem>
        </utu:TabBar>
    </Grid>

</Grid>
```

### Key Rules

Both navigation controls MUST use the same `Region.Name` values so they navigate to the same content:

```xml
<!-- CORRECT: matching Region.Name values across both controls -->
<NavigationViewItem uen:Region.Name="Home" />   <!-- desktop -->
<utu:TabBarItem uen:Region.Name="Home" />        <!-- mobile -->
```

```xml
<!-- WRONG: mismatched names — switching layouts breaks navigation state -->
<NavigationViewItem uen:Region.Name="HomePage" />
<utu:TabBarItem uen:Region.Name="Home" />
```

---

## ResponsiveExtension Approach

**Impact**: MEDIUM — A more concise alternative using Uno Toolkit's markup extension. Requires `Toolkit` in UnoFeatures.

```xml
xmlns:utu="using:Uno.Toolkit.UI"

<!-- Show TabBar only on narrow screens -->
<utu:TabBar uen:Region.Attached="True"
            Visibility="{utu:Responsive Normal=Collapsed, Narrow=Visible}"
            Style="{StaticResource MaterialBottomTabBarStyle}">
    <utu:TabBarItem uen:Region.Name="Home" Content="Home"
                    Style="{StaticResource MaterialBottomTabBarItemStyle}" />
    <utu:TabBarItem uen:Region.Name="Search" Content="Search"
                    Style="{StaticResource MaterialBottomTabBarItemStyle}" />
</utu:TabBar>

<!-- Show NavigationView on normal and wide screens -->
<NavigationView uen:Region.Attached="True"
                Visibility="{utu:Responsive Narrow=Collapsed, Normal=Visible}">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" uen:Region.Name="Home" />
        <NavigationViewItem Content="Search" uen:Region.Name="Search" />
    </NavigationView.MenuItems>
    <Frame uen:Region.Attached="True" />
</NavigationView>
```

### ResponsiveExtension Default Breakpoints

| Name | Default Width |
|---|---|
| Narrowest | 0px |
| Narrow | 0px |
| Normal | 641px |
| Wide | 1008px |
| Widest | 1920px |

Custom breakpoints can be set via `ResponsiveHelper`:

```csharp
// In App.xaml.cs or startup
ResponsiveHelper.SetBreakpoints(new ResponsiveBreakpoints
{
    Narrow = 0,
    Normal = 700,
    Wide = 1000
});
```

---

## Vertical TabBar for Wide Screens

**Impact**: MEDIUM — Alternative to NavigationView for wide screens that keeps TabBar consistency across breakpoints.

```xml
<!-- Wide: vertical TabBar as sidebar -->
<utu:TabBar uen:Region.Attached="True"
            Visibility="{utu:Responsive Narrow=Collapsed, Wide=Visible}"
            Style="{StaticResource MaterialVerticalTabBarStyle}">
    <utu:TabBarItem uen:Region.Name="Home" Content="Home">
        <utu:TabBarItem.Icon>
            <SymbolIcon Symbol="Home" />
        </utu:TabBarItem.Icon>
    </utu:TabBarItem>
    <utu:TabBarItem uen:Region.Name="Search" Content="Search">
        <utu:TabBarItem.Icon>
            <SymbolIcon Symbol="Find" />
        </utu:TabBarItem.Icon>
    </utu:TabBarItem>
</utu:TabBar>

<!-- Narrow: bottom TabBar -->
<utu:TabBar uen:Region.Attached="True"
            Visibility="{utu:Responsive Narrow=Visible, Wide=Collapsed}"
            Style="{StaticResource MaterialBottomTabBarStyle}">
    <utu:TabBarItem uen:Region.Name="Home" Content="Home"
                    Style="{StaticResource MaterialBottomTabBarItemStyle}" />
    <utu:TabBarItem uen:Region.Name="Search" Content="Search"
                    Style="{StaticResource MaterialBottomTabBarItemStyle}" />
</utu:TabBar>
```

---

## Common Mistakes

- **Mismatched Region.Name values**: TabBar and NavigationView must use identical `Region.Name` strings for each destination. Case matters.
- **Both controls visible simultaneously**: Only one navigation control should be visible at a time. If both render, navigation behavior is unpredictable.
- **Missing Region.Attached on root Grid**: The outermost container must have `uen:Region.Attached="True"` for any child navigation to work.
- **NavigationView using Visibility navigator**: Even in responsive shells, NavigationView MUST use Frame — not Grid with `Region.Navigator="Visibility"`.
- **Forgetting initial Visibility state**: The control that should be hidden at startup must start `Visibility="Collapsed"`. VisualState triggers only fire on width changes.

---

## Reference

- [Responsive Shell Navigation](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/ResponsiveShell.html)
- [ResponsiveExtension](https://platform.uno/docs/articles/external/uno.toolkit.ui/doc/helpers/responsive-extension.html)
- [NavigationView Regions](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/NavigationView.html)
- [TabBar Regions](https://platform.uno/docs/articles/external/uno.extensions/doc/Learn/Navigation/Advanced/TabBar.html)
