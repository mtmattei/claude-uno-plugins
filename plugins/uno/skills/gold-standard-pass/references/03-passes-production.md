# Production passes - Maturation, Hardening, Content Reality, Production Readiness

These four move the work from "demonstrates the idea" to "could ship on Monday". They
run after composition and hierarchy are settled, because real data and real states will
stress whatever those passes decided.

---

## Pass 6 - Maturation

**Goal:** remove prototype residue.

Signs to hunt for:

- placeholder-shaped layouts - a region that exists to be filled later
- oversimplified interactions: one happy path, no branches
- idealized content: names of even length, numbers that all fit, dates in one format
- excessively clean data - no nulls, no zeros, no outliers
- missing secondary states - selected, disabled, partial, stale
- generic labels: "Submit", "Item", "Details", "Info", "Card 1"
- weak hierarchy inherited from a wireframe
- components that appear isolated rather than part of a system
- UI that shows the idea and has never been lived in

Fixes:

- **Label rewrite.** Plain verbs from the user's side of the screen. An action keeps its
  name through the whole flow: a button that says "Start route" leads to a screen that
  says "Route started", never "Trip initiated". Copy is design material and gets the
  same intentionality as spacing.
- **Populate with plausible data**, then fix whatever breaks (this feeds Content Reality).
- **Add the second and third case.** One row is a mock; a list with a long row, a short
  row, and an unusual row is a product.
- **Explain computed results.** A price shows its formula; a score shows its inputs. No
  unexplained automation.

---

## Pass 8 - Hardening

**Goal:** survive real usage rather than the screenshot.

Test - mentally where cheap, concretely where possible - against:

| Condition | What to check |
|---|---|
| Long text | Wrapping, truncation, ellipsis placement, tooltip on truncation |
| Short text | Does the layout collapse or look empty |
| Missing data | Placeholder, dash, or hidden - chosen deliberately and applied consistently |
| Large numbers | Column width, thousands separators, tabular figures so digits do not jitter |
| Zero values | "0" reads differently from empty; both need a decision |
| Unexpected values | Negatives, futures dates, over-100% |
| Empty states | Present, and offering the next action |
| Loading | Present; skeleton preserves the populated layout's geometry |
| Errors | What happened, and what to do about it |
| Slow operations | Progress or optimistic feedback past ~400ms |
| Overflow | Horizontal scroll containers, wrap behaviour, min widths |
| Screen sizes | Narrowest supported width and widest realistic one |
| Dense content | 200 rows, not 6 |
| Sparse content | 1 row |
| Localization | Longer strings (German runs ~30% longer), date and number formats, RTL if supported |
| Accessibility | Contrast on the chosen canvas, status never color-alone, focus visible, semantic names |
| Keyboard | Full traversal, visible focus, Enter/Escape on dialogs |
| Touch targets | 44x44 minimum on touch surfaces |

Building the Loading / Empty / Error states of a component is `uno-component-states`'
job - it preserves the populated component's structure, spacing, typography, shape and
density. Invoke it rather than hand-rolling states here.

Reduced motion, focus visuals, and virtualization correctness in XAML belong to
`userinterface-wiki-uno`.

---

## Pass 9 - Content Reality

**Goal:** the UI stays convincing when content stops being convenient.

Replace every convenient string and number with a plausible production one, then look
again:

- label lengths - mix short and long in the same list
- names - including one very long one and one with punctuation or accents
- dates - relative and absolute, including today, yesterday, and last year
- numbers and currency - large, negative, fractional, zero
- counts - 0, 1, 999+
- empty data
- long descriptions that must wrap or truncate
- mixed-content lengths inside one repeated component

Then check truncation and wrapping specifically: where the ellipsis falls, whether the
important half survives, and whether a truncated string is reachable in full.

The test: if a screenshot with realistic content looks worse than the one with mock
content, the layout was tuned to the mock. Fix the layout, not the content.

---

## Pass 11 - Production Readiness

**Goal:** verify it could genuinely ship. This is a checklist pass; findings become work
or become explicit non-goals.

- [ ] All major states exist for every component that has them
- [ ] Components behave consistently across the surface
- [ ] Responsive behaviour is credible at the supported range, not just two breakpoints
- [ ] Accessibility considered: contrast, focus, semantics, target size, color-alone
- [ ] Interactions are implementable on the target platform
- [ ] Layouts tolerate real data (Pass 9 signed off)
- [ ] Controls have predictable behaviour - no control that does something different here
- [ ] Error recovery exists where an error is possible - retry, undo, or a way out
- [ ] Empty states give useful direction rather than an apology
- [ ] Loading states are handled, including the slow case
- [ ] Destructive actions are treated appropriately: confirmation, undo, or both, and
      never styled as the visual default
- [ ] Nothing important is implicit

For a running Uno app, tick these against the running app, not the markup - launch and
inspect via `uno-verify`. A readiness checklist signed off from source is a guess.

Anything that cannot ship is written down as an Unresolved Question with what it would
take, rather than quietly left off the list.
