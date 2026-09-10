# Structure passes - Refinement, Resolution, Coherence

These three fix *what the design already is*. They run early because every later pass
inherits their output. None of them changes the visual language.

---

## Pass 1 - Refinement

**Goal:** improve the execution of decisions that are already correct.

The underlying idea is right; the numbers are approximate. Sharpen them.

Examine:

- spacing and the spacing scale in use
- alignment - shared edges, shared baselines, shared gutters
- sizing and proportion of components against each other
- typography: size steps, weight steps, line height, measure (line length)
- layout relationships - what sits next to what, and how far apart
- visual rhythm and whitespace distribution
- control placement within a component

How to work it:

1. Extract the scale actually in use. List every distinct padding, margin, gap, radius,
   font size, and icon size on the surface, with counts.
2. Near-duplicates are the finding. `12 / 13 / 14` across three siblings is one value
   with two typos. Collapse to the dominant value unless the difference is deliberate.
3. Set a real scale if none exists (a 4pt or 8pt grid is the default; the existing
   dominant values decide which). Do not import a scale from another product.
4. Check measure on body text - roughly 45-75 characters. Long-measure paragraphs read
   as unstyled far more than any font choice.

Do not reinterpret the design. If a value looks odd but is consistent and intentional,
it stays.

**Output shape:** `<element> <property> <old> to <new> - <reason>`.

---

## Pass 2 - Resolution

**Goal:** turn unfinished or ambiguous decisions into deliberate ones.

Find:

- awkward spacing decisions that read as "whatever the container did"
- unclear grouping - items that belong together but are not visually bound
- half-resolved layouts: a column that stops mid-way, an area with no defined job
- inconsistent hierarchy inside one component
- controls that look temporarily placed - a button parked in the corner because it had
  to go somewhere
- unclear information priority: three elements competing to be first
- unresolved component behaviour - what happens on tap, on overflow, when empty
- elements that appear arbitrary or default: stock accent color, untouched control
  chrome, a default corner radius on one thing and a chosen one on the rest

The diagnostic question for each: **could I say out loud why this is the way it is?**
If the answer is "it was already like that", it is unresolved.

Resolving means choosing, then applying the choice consistently. It does not mean
adding a container to hide the ambiguity - see *Moves that are not polish* in SKILL.md.

Common resolutions, in preference order:

1. Move it (position carries priority for free)
2. Change its size or weight
3. Change its spacing or grouping
4. Change its contrast
5. Only then, give it a surface of its own

---

## Pass 3 - Coherence

**Goal:** make the product feel like one system rather than a set of screens.

Harmonize across the whole surface, and across sibling surfaces if they are in scope:

- spacing values
- component dimensions - control heights, card widths, row heights
- typography roles: the same kind of text uses the same style everywhere
- icon sizing and stroke weight
- corner treatment - one radius family, with deliberate exceptions
- control behaviour - the same control does the same thing everywhere
- alignment and layout conventions - where titles sit, where actions sit
- interaction patterns - selection, dismissal, confirmation
- repeated UI structures - list rows, cards, headers, toolbars

Method:

1. Inventory the repeats. Every card variant, every row variant, every button size.
2. For each family, pick the canonical version - normally the one used most, or the one
   used in the most important place.
3. Fold the variants in. Record each fold as a change with its reason.
4. Keep every difference that carries meaning, and say what the meaning is.

**The guard:** mechanical uniformity is a failure, not a success. If Hierarchy or Art
Direction introduced a deliberate contrast - a larger primary card, a heavier total
row, a wider hero column - Coherence must not average it away. Per the no-undo rule in
SKILL.md, reversing an earlier pass's deliberate decision requires stating it.

Similar things behave similarly. Different things differ on purpose.

**Platform note (XAML):** coherence work usually means promoting repeated literals into
`x:Double` / `Thickness` / `CornerRadius` resources in the app dictionary, then binding
components to them. Custom keys inside `ThemeDictionaries` resolve unreliably on Uno -
use a plain root-level dictionary merged in `App.xaml`. Material role overrides stay in
`ColorPaletteOverride.xaml`. Mechanics for both live in `uno-material`.
