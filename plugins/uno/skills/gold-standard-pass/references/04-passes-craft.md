# Craft passes - Interaction, Fit & Finish

These two are the last layer before the Quality Bar gate. They assume composition,
hierarchy, and content are settled; both of them are cheap to redo and expensive to do
early.

---

## Pass 7 - Interaction

**Goal:** the interface behaves like a finished product, not a mockup with click targets.

Review the states that apply to each interactive element:

`default / hover / focus / pressed / selected / disabled / loading / success / warning /
error / dragging / expanding / collapsing`

Improve:

- **Feedback** - every action acknowledges itself within ~100ms, even if the result takes
  longer
- **Affordance** - interactive things look interactive at rest, without hover
- **Transitions** - state changes are continuous rather than teleported
- **Discoverability** - a feature reachable only by hover is not discoverable on touch
- **Perceived responsiveness** - optimistic updates, skeletons that match final geometry,
  progress past ~400ms

Checks that catch the common gaps:

- Focus is visible and distinct from hover. Keyboard users get a real ring, not a tint.
- Disabled says why, or is not disabled. A dead button with no explanation is a bug.
- Pressed state exists on touch surfaces - hover does not exist there.
- Selection is durable and obvious after the pointer leaves.
- Destructive actions get a distinct treatment and a way back.
- The same gesture means the same thing everywhere on the surface.

**Motion numbers are not decided here.** Durations, easing curves, press scale, spring
constants, stagger, and reduced-motion gating come from `xaml-design-polish` for XAML
and `web-interface-polish` for web. Invoke the owning skill and cite the exact values
being applied. House values differ by platform - XAML press scale is 0.98, web is 0.96.

Constructing Loading / Empty / Error states for a component is `uno-component-states`.

Restraint is the standard: interactions feel deliberate, predictable, and quiet. An
animation that draws attention to itself twice has failed.

---

## Pass 10 - Fit & Finish

**Goal:** the final 10% of craftsmanship. This is where "finished" actually comes from.

Inspect:

- **Optical alignment** - a triangle, a circle, and a square at the same coordinate do not
  look aligned. Icons in circular buttons usually need a fraction of a pixel of offset;
  a play glyph needs more. Trust the eye over the coordinate.
- **Baselines** - text next to text should sit on a shared baseline, not a shared box
- **Internal padding** - equal on the axes it should be equal on, and optically equal
  where a glyph's sidebearing lies about it
- **Icon and text relationships** - cap-height matched, gap consistent, both vertically
  centred as a unit
- **Small inconsistencies** - one radius, one border weight, one shadow, one opacity value
  that differs from its family for no reason
- **Border weights** - hairlines that render at 1.5px on fractional scaling; borders that
  are one value on one control and another elsewhere
- **Component proportions** - control heights against each other. Material sizes ComboBox
  at 56px, towering over 37px token fields; a property-setter style fixes it
- **Text wrapping** - orphans and widows in headings, a two-word second line
- **Rhythm** - vertical spacing that repeats predictably down the page
- **Micro-interactions** - the small acknowledgements, tuned per the owning motion skill
- **Transition timing** - values from the motion skill, applied consistently
- **Alignment across neighbours** - edges shared between adjacent components, gutters that
  line up across regions

Method: work at 100% zoom for judgment, at 400% for alignment, and at thumbnail size for
rhythm. Then look at the surface upside down or mirrored - misalignment survives the
flip, familiarity does not.

**Polish is not decoration.** Nothing on the forbidden-moves list in SKILL.md becomes
acceptable because it is Fit & Finish. If the finding is "this area feels weak", it is a
Hierarchy or Composition finding that arrived late, and it goes back to that pass.
