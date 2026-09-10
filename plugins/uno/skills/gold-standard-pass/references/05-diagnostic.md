# Diagnostic - preservation inventory and symptom-to-pass lookup

Read this during Step 1, before any change.

---

## Part 1 - Preservation inventory

Write down what is being preserved before deciding what to improve. Anything not on this
list is fair game; anything on it needs a stated reason to touch.

```
Palette:        <named values, and what each one means>
Type:           <faces and the roles they play>
Shape:          <radius family, border weights, elevation approach>
Density:        <tight / regular / airy, and where it varies>
Layout concept: <the archetype - rail, board, ledger, feed, canvas>
Signature:      <the one element this surface is remembered by>
Interaction:    <the major model - direct manipulation, form, wizard, browse>
Identity:       <what makes it recognizably this product>
```

If a field cannot be filled in, the design has no established decision there. That is a
**Resolution** finding, not permission to invent one from a different vocabulary - derive
it from the fields that *are* filled in.

If the inventory itself is the problem - the direction is generic, templated, or wrong for
the subject - stop. That is `xaml-art-direction` work and it needs the user's go-ahead,
because it breaks the prime directive.

---

## Part 2 - Symptom to pass

| What you observe | Pass |
|---|---|
| Values near-duplicate each other (12/13/14 padding) | 1 Refinement |
| Long line lengths, cramped line height, odd type steps | 1 Refinement |
| Elements almost line up | 1 Refinement, then 10 Fit & Finish |
| "Why is this here?" has no answer | 2 Resolution |
| Stock accent color, untouched control chrome | 2 Resolution |
| A control parked wherever it fit | 2 Resolution |
| An area with no defined job | 2 Resolution |
| Three card variants doing the same thing | 3 Coherence |
| The same action styled differently in two places | 3 Coherence |
| Icons at three sizes and two stroke weights | 3 Coherence |
| Everything survives equally at thumbnail size | 4 Art Direction |
| Uniform margins on every edge of every region | 4 Art Direction |
| Layout matches the default cluster (band + cards + rail) | 4 Art Direction |
| Regions separated by cards where space would do | 4 Art Direction |
| Two equal-weight primary buttons | 5 Hierarchy |
| Metadata as loud as content | 5 Hierarchy |
| Cannot tell what the screen is about in one second | 5 Hierarchy |
| Status carried by color alone | 5 Hierarchy, and 8 Hardening |
| Labels like "Submit", "Item", "Details" | 6 Maturation |
| Every name the same length, every number fits | 6 Maturation, 9 Content Reality |
| Only the happy path exists | 6 Maturation |
| No selected / disabled / stale states | 6 Maturation, 7 Interaction |
| No hover, focus, or pressed feedback | 7 Interaction |
| Focus ring missing or identical to hover | 7 Interaction |
| Disabled with no explanation | 7 Interaction |
| Instant state teleports, no transition | 7 Interaction (numbers from the motion skill) |
| Breaks with a long string or a big number | 8 Hardening |
| No empty / loading / error state | 8 Hardening, built via `uno-component-states` |
| Untested at the narrowest width | 8 Hardening |
| Touch targets under 44px | 8 Hardening |
| Screenshot looks worse with real data | 9 Content Reality |
| Truncation cuts the important half | 9 Content Reality |
| Destructive action styled as the default | 11 Production Readiness |
| No way back from an error | 11 Production Readiness |
| Icon looks off-centre despite correct coordinates | 10 Fit & Finish |
| Hairline renders inconsistently | 10 Fit & Finish |
| Control heights disagree in one row | 10 Fit & Finish |
| Heading wraps to a two-word second line | 10 Fit & Finish |

---

## Part 3 - Scoping the run

Count the findings, then pick the scope:

| Findings | Scope |
|---|---|
| 1-3, all in one pass | Run that pass alone, plus the Quality Bar gate |
| Several, all cosmetic | Refinement + Fit & Finish + gate |
| Priority unclear or composition weak | Hierarchy + Art Direction first, then reassess |
| Falls over on real data | Content Reality + Hardening, visual layer untouched |
| Broadly prototype-shaped | Full Gold Standard Pass in the Step 3 order |
| None material | Say so. "At bar, nothing warranted" is a complete answer |

The last row is real and gets used. Manufacturing findings to justify a run is the
failure this skill exists to prevent.

---

## Part 4 - Diagnosis output template

```
## Diagnosis

Preserving:
- Palette: ...
- Type: ...
- Layout concept: ...
- Signature: ...

Findings:
- <symptom> - <file:line or element> - Pass <n>
- ...

Passes warranted: <n, n, n> (run order per SKILL.md Step 3)
Passes skipped: <n> - <reason>; <n> - <reason>
Estimated change footprint: <n files, n edits>
```

Present the diagnosis before editing when the run is broad (four or more passes), so the
user can narrow it. For one- or two-pass runs, diagnose and proceed.
