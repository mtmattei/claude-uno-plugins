---
name: wpf-migration-assessment
description: "Assess WPF projects for Uno Platform migration readiness. Analyzes codebase structure, dependencies, controls, patterns, and platform coupling to produce a scored assessment with effort estimate, risk level, blockers, and recommended migration path. Use when: (1) Evaluating whether a WPF project is a good migration candidate, (2) Estimating migration effort and risk for planning, (3) Producing a technical due-diligence report for a WPF-to-Uno migration, (4) Triaging multiple WPF apps to decide migration priority, (5) Creating a migration proposal for stakeholders. Do NOT use for: Executing the actual migration (see wpf-to-uno-migration), new Uno project setup (see uno-platform-agent), or Silverlight migration (see silverlight-to-uno-migration)."
license: "Apache 2.0"
metadata:
  version: "1.0.0"
  author: "Uno Platform"
  category: "uno-platform-migration"
  tags: [wpf, migration, assessment, planning, uno-platform]
---

# WPF Migration Assessment

Systematic assessment framework for evaluating WPF projects for Uno Platform migration. Produces a scored report with effort estimate, risk classification, blockers, and recommended migration strategy.

## Assessment Workflow

When this skill is invoked, follow these steps in order:

### Step 1: Codebase Inventory

Analyze the WPF project and collect these metrics:

```
INVENTORY CHECKLIST:
[ ] Total solution projects (count)
[ ] Total .cs files (count)
[ ] Total .xaml files (count)
[ ] Estimated lines of code
[ ] .NET version / target framework
[ ] Project format (SDK-style vs classic .csproj)
[ ] NuGet packages (list all)
[ ] Project references and dependencies
[ ] License type
```

**How to collect:**
- Use `Glob` to find all `.cs`, `.xaml`, `.csproj` files
- Read `.csproj` files for target framework and package references
- Read `packages.config` if classic format
- Check for `global.json`, `Directory.Build.props`

### Step 2: Architecture Analysis

Classify the codebase architecture:

| Pattern | Detection Method | Impact |
|---------|-----------------|--------|
| MVVM | ViewModels folder, INotifyPropertyChanged, ICommand | LOW — highly portable |
| Code-behind | Logic in .xaml.cs files, event handlers | MEDIUM — works but less portable |
| Prism | PrismApplication, RegionManager | MEDIUM — needs navigation rework |
| MVVM Light | ViewModelLocator, SimpleIoc | LOW — replace with CommunityToolkit.Mvvm |
| ReactiveUI | ReactiveObject, WhenAnyValue | LOW — cross-platform library |
| No pattern | Mixed concerns, singletons, static state | HIGH — needs refactoring |

### Step 3: Dependency Classification

For each NuGet package or library, classify as:

| Classification | Meaning | Action |
|---------------|---------|--------|
| **Portable** | .NET Standard / cross-platform | Keep unchanged |
| **Replaceable** | WPF-specific with Uno equivalent | Map to Uno equivalent |
| **Abstractable** | Platform-specific but can be wrapped | Create interface + platform impl |
| **Blocker** | No cross-platform path | Evaluate workaround or scope reduction |

**Common dependency classifications:**

| Dependency | Classification | Uno Alternative |
|-----------|---------------|----------------|
| CommunityToolkit.Mvvm | Portable | Keep |
| Newtonsoft.Json | Portable | Keep (or System.Text.Json) |
| Entity Framework Core | Portable | Keep |
| Entity Framework 6 | Replaceable | Upgrade to EF Core |
| Microsoft.Extensions.DI | Portable | Keep |
| System.Reactive | Portable | Keep |
| MahApps.Metro | Replaceable | Uno Material + Uno Toolkit |
| MaterialDesignInXaml | Replaceable | Uno Material |
| iNKORE.UI.WPF.Modern | Replaceable | Uno Material |
| Dragablz | Replaceable | TabView |
| AvalonEdit | Blocker/Abstractable | TextBox or WebView2+Monaco |
| CefSharp | Blocker | WebView2 (if simple) or N/A |
| Telerik WPF | Replaceable | Control-by-control mapping |
| DevExpress WPF | Replaceable | Control-by-control mapping |
| SQL Server LocalDB | Replaceable | SQLite |
| WPF-UI (lepoco/wpfui) | Replaceable | Uno Material |
| Microsoft.AspNet.SignalR | Replaceable | Microsoft.AspNetCore.SignalR.Client |
| log4net / NLog | Replaceable | Microsoft.Extensions.Logging |
| Unity (DI) | Replaceable | Microsoft.Extensions.DI |

### Step 4: Control Audit

Count usages of controls by migration difficulty:

**Zero-effort controls** (WinUI equivalent exists with same API):
- Grid, StackPanel, Border, Canvas, TextBlock, TextBox, Button, CheckBox, RadioButton, ComboBox, Slider, Image, ScrollViewer, ListView, ListBox, ItemsControl, UserControl, ContentControl, ToggleButton, PasswordBox, ProgressBar, ProgressRing

**Low-effort controls** (name or API change):
- DatePicker → CalendarDatePicker
- TabControl → TabView or Pivot
- Expander → Expander (WCT)
- ToolTip → ToolTip (same but verify)

**Medium-effort controls** (requires redesign or different package):
- DataGrid → CommunityToolkit DataGrid
- TreeView → TreeView (verify feature parity)
- WebBrowser → WebView2
- Menu/MenuItem → MenuBar + MenuBarItem + MenuFlyoutItem
- ToolBar → CommandBar
- StatusBar → Custom Grid layout

**High-effort / No equivalent:**
- RichTextBox → RichEditBox (limited) or WebView2
- FlowDocument → WebView2 + HTML
- DocumentViewer → WebView2 + PDF.js
- WindowsFormsHost → No equivalent (remove)
- InkCanvas → InkCanvas (verify platform support)

### Step 5: Platform API Audit

Check for usage of these Windows-specific APIs:

| API Category | Detection | Impact |
|-------------|-----------|--------|
| P/Invoke (`DllImport`, `LibraryImport`) | Grep for `DllImport`, `LibraryImport` | HIGH — needs abstraction |
| COM Interop | Grep for `ComImport`, `Marshal.` | HIGH — Windows only |
| Registry | Grep for `Registry`, `RegistryKey` | MEDIUM — needs alternative storage |
| Shell integration | Grep for `Shell32`, `SHFileOperation` | HIGH — platform-specific |
| Windows.Forms reference | Check .csproj for WinForms reference | MEDIUM — remove or abstract |
| Process.Start | Grep for `Process.Start` | LOW — replace with Launcher |
| Clipboard (WPF) | Grep for `System.Windows.Clipboard` | LOW — namespace change |
| Printing | Grep for `PrintDialog`, `PrintDocument` | MEDIUM — platform-specific |
| Notification area / tray | Grep for `NotifyIcon`, `TaskbarIcon` | HIGH — platform-specific |
| Global hotkeys | Grep for `RegisterHotKey` | HIGH — platform-specific |

### Step 6: XAML Feature Audit

Check for WPF XAML features NOT supported in Uno:

| Feature | Detection | Impact | Replacement |
|---------|-----------|--------|-------------|
| `x:Static` | Grep XAML for `x:Static` | MEDIUM | StaticResource |
| `MultiBinding` | Grep XAML for `MultiBinding` | MEDIUM | Run elements or computed property |
| `StringFormat` in Binding | Grep XAML for `StringFormat` | MEDIUM | Converter or computed property |
| `Style.Triggers` | Grep XAML for `Style.Triggers` | HIGH | VisualStateManager |
| `DataTrigger` | Grep XAML for `DataTrigger` | HIGH | VisualStateManager + code |
| `EventTrigger` | Grep XAML for `EventTrigger` | MEDIUM | Code-behind or behaviors |
| `PropertyTrigger` | Grep XAML for `Trigger Property` | HIGH | VisualStateManager |
| `RelativeSource AncestorType` | Grep XAML for `AncestorType` | MEDIUM | x:Bind or ElementName |
| `d:DesignInstance` | Grep XAML for `d:DesignInstance` | LOW | Remove (design-time only) |

Count occurrences of each. High counts of Triggers = significant XAML rework.

### Step 7: Scoring

Score the project across 6 dimensions:

| Dimension | Weight | Score Range | How to Score |
|-----------|--------|-------------|-------------|
| **Architecture** | 25% | 1-5 | 5=clean MVVM+DI, 4=MVVM no DI, 3=code-behind, 2=mixed, 1=spaghetti |
| **Dependencies** | 20% | 1-5 | 5=all portable, 4=1-2 replaceable, 3=3-5 replaceable, 2=blockers present, 1=mostly blockers |
| **Controls** | 20% | 1-5 | 5=all zero/low effort, 4=some medium, 3=DataGrid+custom, 2=many high-effort, 1=dominant high-effort |
| **Platform coupling** | 20% | 1-5 | 5=no platform APIs, 4=1-2 abstractable, 3=P/Invoke present, 2=deep shell integration, 1=fundamentally Windows-only |
| **XAML compatibility** | 10% | 1-5 | 5=no unsupported features, 4=few x:Static, 3=MultiBinding+StringFormat, 2=Triggers heavy, 1=many unsupported patterns |
| **Project health** | 5% | 1-5 | 5=SDK-style+.NET 8+, 4=.NET 6/7, 3=.NET Core 3.1, 2=.NET Framework 4.8, 1=.NET Framework <4.5 |

**Overall Score = Weighted Average (1.0 - 5.0)**

### Step 8: Classification

| Score Range | Classification | Meaning |
|:-----------:|:------------:|---------|
| 4.0 - 5.0 | **GREEN — Strong Fit** | Low complexity, strong migration candidate. Expect 1-3 days for small apps, 1-2 weeks for medium. |
| 3.0 - 3.9 | **YELLOW — Manageable** | Moderate complexity, manageable with planning. Expect 1-3 weeks for medium apps. Some areas need redesign. |
| 2.0 - 2.9 | **ORANGE — Challenging** | High complexity, phased migration recommended. Expect 2-6 weeks. Consider Uno Islands for gradual approach. |
| 1.0 - 1.9 | **RED — High Risk** | Fundamental blockers present. Consider partial migration, Uno Islands only, or evaluate if migration is the right path. |

### Step 9: Generate Report

Produce a structured assessment report:

```markdown
# WPF Migration Assessment: [App Name]

## Summary
- **Classification:** [GREEN/YELLOW/ORANGE/RED]
- **Overall Score:** [X.X / 5.0]
- **Estimated Effort:** [X days/weeks]
- **Recommended Strategy:** [Full migration / Phased migration / Partial + Uno Islands / Not recommended]

## Scores
| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Architecture | X/5 | |
| Dependencies | X/5 | |
| Controls | X/5 | |
| Platform coupling | X/5 | |
| XAML compatibility | X/5 | |
| Project health | X/5 | |

## Inventory
- Projects: X
- C# files: X
- XAML files: X
- Estimated LOC: X
- Target framework: X
- License: X

## Blockers
<!-- List any hard blockers -->

## Risk Areas
<!-- List areas requiring significant rework -->

## Migration Path
<!-- Recommended phased approach -->

## Effort Breakdown
| Phase | Estimated Effort | Risk |
|-------|:----------------:|:----:|
| Project setup + namespace conversion | X hrs | LOW |
| Business logic migration | X hrs | LOW |
| XAML / UI migration | X hrs | MED |
| Dependency replacement | X hrs | MED |
| Platform API abstraction | X hrs | HIGH |
| Testing + polish | X hrs | LOW |
| **Total** | **X hrs** | |

## Recommendations
<!-- Strategic recommendations for the migration -->
```

## Effort Estimation Formulas

Use these heuristics after collecting inventory:

```
Base Hours = (XAML files × 0.5) + (C# files with platform APIs × 2)
+ Third-party UI library: add 8-16 hours
+ DataGrid: add 4-8 hours per DataGrid view
+ Multi-window → single-window: add 4-8 hours
+ EF6 → EF Core: add 4-8 hours
+ P/Invoke: add 4-16 hours per P/Invoke group
+ Unsupported XAML features: add 0.5 hours per occurrence
```

## Quick Assessment (Under 5 Minutes)

For a rapid assessment without deep analysis, check these 5 signals:

1. **Is it MVVM?** → If yes, architecture score ≥ 4
2. **Does it use MahApps/MaterialDesign/Telerik?** → If yes, dependencies score ≤ 3
3. **Does it use DataGrid?** → If yes, controls score ≤ 3
4. **Does it have DllImport/P/Invoke?** → If yes, platform coupling score ≤ 2
5. **Is it on .NET 6+?** → If yes, project health score ≥ 4

Two or more "bad" signals → YELLOW or worse.
Four+ "bad" signals → ORANGE or RED.

## Related Skills

| Skill | Use when... |
|---|---|
| `wpf-to-uno-migration` | Ready to execute the migration (assessment is done) |
| `uno-platform-agent` | Creating a new Uno project from scratch |
| `uno-build-troubleshoot` | Fixing build/runtime errors during migration |

## Reference

See [references/scoring-examples.md](references/scoring-examples.md) for scored examples from the WPF Migration 5x campaign.
