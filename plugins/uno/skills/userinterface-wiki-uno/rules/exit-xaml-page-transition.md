---
title: Page Transition Consistency
impact: MEDIUM
tags: exit, navigation, page, frame
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
