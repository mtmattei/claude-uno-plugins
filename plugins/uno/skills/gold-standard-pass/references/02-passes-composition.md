# Composition passes - Art Direction, Hierarchy

These two decide what the eye does. They run right after Resolution, because
composition and priority set the frame every later pass works inside.

Neither pass invents a new visual identity. Choosing palette, type, and signature
element belongs to `xaml-art-direction`, and reaching for it means the prime directive
has been broken - stop and ask.

---

## Pass 4 - Art Direction (within the approved direction)

**Goal:** move the surface from *arranged* to *composed*.

Evaluate:

- what the eye lands on first, second, third - and whether that matches importance
- primary versus secondary emphasis
- balance and optical weight across the surface
- framing: what the outer margins do, where the surface breathes
- negative space as a deliberate element rather than leftover
- scale relationships between neighbouring blocks
- section boundaries - how a region ends and the next begins
- grouping and density: which areas are dense on purpose
- the composition as a whole, seen squinting

The question: **does this look composed, or merely arranged?**

Working method:

1. Squint at it, or render it small. Whatever survives at thumbnail size is the
   composition. If everything survives equally, there is no composition.
2. Name the intended focal point. One per surface.
3. Check the weight map: which quadrant is heaviest, and is that on purpose.
4. Look at the outer frame. Uniform 16px margins on every edge of every region is the
   default nobody chose - top and bottom rarely want the same value as left and right.
5. Check the boundaries between regions. Separation carried by space reads more designed
   than separation carried by a line, and a line reads more designed than a card.

Levers available, in preference order: **space, scale, alignment, density, position**.
Drawn separators come after all five.

**Composition default cluster** - if the layout matches this, it was inherited, not
composed (validated across repeated generated builds): identity band across the top with
one filled primary top-right, card list on the left two-thirds, stacked rail on the
right third, three-segment nav on narrow, and inside every card a big figure left plus a
status chip top-right plus text actions along the bottom. Matching it is a finding.

Preserve the visual language throughout. Composition changes move and size existing
elements; they do not restyle them.

---

## Pass 5 - Hierarchy

**Goal:** make information priority understandable in the first second.

Improve the relationship between:

- primary actions vs secondary vs tertiary
- titles and section headings
- supporting information
- metadata
- navigation
- content
- status
- controls

Use, in this order:

1. **Position** - first in reading order, or in the optically dominant slot
2. **Scale** - a genuine step, not 2px
3. **Weight** - one or two steps on the type ramp
4. **Contrast** - ink strength against the canvas
5. **Spacing** - proximity binds, distance separates
6. **Grouping** - a shared container is the last resort, not the first

Never solve hierarchy with decoration. A colored left border, a badge, or a new card is
what gets reached for when position, scale, and spacing were not tried.

Checks:

- **One primary action per view.** Two filled buttons of equal weight means neither is
  primary. Demote one to outline or text.
- **Three levels, at most, in one component.** Title, value, meta. A fourth level is
  usually two components sharing a box.
- **Metadata should be quiet.** Timestamps, IDs, and counts at full ink weight compete
  with content.
- **Status is meaning, not decoration.** A semantic color that also decorates has stopped
  being semantic. Status must never be carried by color alone - pair it with text, shape,
  or icon.
- **Scan order matches priority.** Read the surface top-left to bottom-right and list
  what registers. If a secondary element registers before the primary one, that is the
  finding.
- **Empty-ish states keep the hierarchy.** A screen with two rows must still show which
  element is primary.

**Output shape:** state the intended priority order first, then each change that enforces
it, in numbers - "total row 14 to 20, weight 500 to 700, ink `#2F6B3C`; label row dropped
to 60% ink so the total leads".
