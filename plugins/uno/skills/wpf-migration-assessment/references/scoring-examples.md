# Scoring Examples — WPF Migration 5x Campaign

Five real-world WPF projects assessed using the WPF Migration Assessment framework. These examples calibrate the scoring system.

---

## Example 1: CalculatorWpfMVVM — GREEN (4.7/5.0)

**Classification:** GREEN — Strong Fit
**Effort:** ~2 hours

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Architecture | 5 | Clean MVVM with ViewModelBase, DelegatingCommand, data binding |
| Dependencies | 5 | Only mXparser (math library, .NET Standard) — fully portable |
| Controls | 5 | Grid, Button, TextBlock, TextBox — all zero-effort |
| Platform coupling | 5 | No platform APIs used |
| XAML compatibility | 4 | Standard bindings, no unsupported features; verify converters |
| Project health | 3 | .NET Framework 4.6.1, classic .csproj (needs SDK conversion) |

**Key takeaway:** The simplest possible migration. Near-zero risk. Proves the path works.

---

## Example 2: NotepadWPF — GREEN (4.2/5.0)

**Classification:** GREEN — Strong Fit
**Effort:** ~6 hours

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Architecture | 3 | Code-behind (not MVVM), but small and well-organized |
| Dependencies | 5 | No third-party packages |
| Controls | 4 | Menu→MenuBar, StatusBar→custom Grid; TextBox is zero-effort |
| Platform coupling | 4 | File dialogs (abstractable), Print (defer), Settings (replace) |
| XAML compatibility | 5 | Standard WPF XAML, no unsupported features |
| Project health | 5 | .NET 6.0, SDK-style project (best starting point) |

**Key takeaway:** Code-behind apps migrate fine. The .NET 6 project format saves time.

---

## Example 3: Reservoom — GREEN (4.5/5.0)

**Classification:** GREEN — Strong Fit
**Effort:** ~8 hours

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Architecture | 5 | Clean MVVM with Stores, Commands, DI, Services |
| Dependencies | 5 | EF Core + SQLite + MS.Extensions.DI — all portable |
| Controls | 4 | ListView, TextBox, DatePicker, Button — all zero/low effort; no DataGrid |
| Platform coupling | 5 | No platform APIs |
| XAML compatibility | 4 | Standard bindings; DatePicker→CalendarDatePicker name change |
| Project health | 4 | Modern .NET with EF Core (verify specific version) |

**Key takeaway:** The ideal LOB migration candidate. EF Core + DI + MVVM = smooth path. Business logic migrates unchanged.

---

## Example 4: SignalChat — YELLOW (3.4/5.0)

**Classification:** YELLOW — Manageable with Planning
**Effort:** ~12 hours

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Architecture | 4 | MVVM with service interfaces, Unity DI, clean separation |
| Dependencies | 3 | Legacy SignalR (replace), MahApps (replace), MaterialDesign (replace), Unity (replace) |
| Controls | 4 | ListView, TextBox, Button, Image — mostly zero-effort |
| Platform coupling | 4 | Dispatcher (abstractable), no P/Invoke |
| XAML compatibility | 3 | MaterialDesign styles need full replacement; Interactivity behaviors |
| Project health | 2 | .NET Framework 4.6, classic .csproj, legacy packages |

**Key takeaway:** Good architecture but heavy on replaceable dependencies. The service interface pattern makes dependency swapping clean. SignalR upgrade is a separate concern from UI migration.

---

## Example 5: Flow Launcher — ORANGE (2.5/5.0)

**Classification:** ORANGE — Challenging, Partial Migration Recommended
**Effort:** 2-3 weeks (partial migration)

| Dimension | Score | Notes |
|-----------|:-----:|-------|
| Architecture | 3 | Mixed MVVM/code-behind, plugin host system, complex |
| Dependencies | 2 | iNKORE.UI.WPF.Modern (replace), Everything SDK (blocker), Shell APIs |
| Controls | 3 | Standard controls but complex custom UI (search bar, results list, plugins) |
| Platform coupling | 1 | Deep shell integration: tray icon, global hotkeys, file indexing, app launching |
| XAML compatibility | 3 | Modern WPF patterns but iNKORE styles throughout |
| Project health | 4 | .NET 7+, SDK-style project |

**Key takeaway:** The core UI and plugin contracts are portable, but shell integration is fundamentally Windows-only. Partial migration (UI + plugin system) or Uno Islands (gradual embedding) is the realistic path. This is the "honest story" — not everything fully migrates, and knowing when to use Uno Islands matters.

---

## Projects That Failed Initial Selection

These WPF projects were evaluated but disqualified before scoring:

### MarkDownEditor — DISQUALIFIED (License + Hard Blocker)

| Issue | Severity |
|-------|----------|
| GPL-3.0 license | Cannot use for public campaign case study |
| CefSharp dependency (Chromium Embedded Framework) | Native Win32 browser, fundamentally non-portable |
| Bundled native DLLs (wkhtmltox, pandoc) | Platform-specific binaries |

**Lesson:** Always check license AND native dependencies before investing in assessment.

### Red-Inventory-Management — DISQUALIFIED (No License)

| Issue | Severity |
|-------|----------|
| No LICENSE file in repository | Cannot legally fork or publicly reference |
| SQL Server LocalDB dependency | Windows-only, needs replacement |
| Good MVVM architecture wasted by legal blocker | Would have scored ~3.5 (YELLOW) |

**Lesson:** No license = cannot proceed. Contact the author before investing time.

### Dependencies (lucasg) — DISQUALIFIED (Fundamentally Non-Portable)

| Issue | Severity |
|-------|----------|
| C++/CLI interop layer (ClrPhlib) | Requires native C compilation |
| ProcessHacker native libraries | Windows kernel-level functionality |
| PE file analysis is inherently Windows-only | No cross-platform use case |
| Code-behind heavy (not MVVM) | Adds to migration cost |

**Lesson:** When the entire domain is platform-specific (DLL analysis = Windows), migration doesn't make sense. The app's purpose only exists on Windows.

---

## Score Distribution Across 5x Campaign

```
5.0 ┤
4.5 ┤ ●Calculator(4.7)  ●Reservoom(4.5)
4.0 ┤ ●Notepad(4.2)
3.5 ┤ ●SignalChat(3.4)
3.0 ┤
2.5 ┤ ●FlowLauncher(2.5)
2.0 ┤
1.5 ┤
1.0 ┤
    └──────────────────────────────────
     GREEN        YELLOW      ORANGE    RED
```

**Distribution insight:** 3 of 5 projects score GREEN, 1 scores YELLOW, 1 scores ORANGE. This matches the expected real-world distribution: **most WPF apps are migratable**, but complex/shell-integrated apps need a different strategy.
