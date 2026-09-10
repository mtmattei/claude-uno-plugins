---
name: gold-standard-pass
description: Bring an existing design, prototype, or interface to production-grade quality without redesigning it - the approved direction is preserved and only the execution is elevated. Diagnoses first, then runs only the warranted passes out of twelve (Refinement, Resolution, Coherence, Art Direction, Hierarchy, Maturation, Interaction, Hardening, Content Reality, Fit & Finish, Production Readiness, Quality Bar), always ending on the Quality Bar gate. Use when the user says "gold standard pass", "polish this", "make it production-ready", "make it feel finished", "level this up", "tighten this up", "it still feels prototype-y / unfinished / generic / assembled", "fit and finish", "harden this", or names any single pass. Do NOT use for a redesign or a new visual identity (see xaml-art-direction); motion numbers come from xaml-design-polish (XAML) or web-interface-polish (web); loading/empty/error states from uno-component-states; XAML per-detail correctness from userinterface-wiki-uno; code cleanup from uno-audit.
---

# Gold Standard Pass

Take work that already exists - a prototype, a screen, a component, a shipped-but-rough product - and close the gap between it and a polished, intentional, production-ready result.

**Resolve, don't redesign. Elevate, don't restyle.**

The current design is the approved art direction. Treat it as a client decision that has already been signed off. The job is execution quality, not a new concept.

## The prime directive

Preserve, without negotiation:

- the existing visual language (palette, type, shape, density)
- the core layout concept
- product identity and established aesthetic
- the major interaction model
- recognizable component patterns
- the original design intent

Improve anything that reads as prototype-level, generic, accidental, inconsistent, unresolved, visually weak, mechanically assembled, incomplete, unpolished, hard to use, or unrealistic in production.

**Never introduce change merely to make the design look different.** Every modification carries a reason, and that reason goes in the change list.

## Role split - this skill orchestrates, it does not duplicate

| Question | Owner |
|---|---|
| What should this design *be*? New identity, palette, signature element | `xaml-art-direction` (XAML) - and this skill stands down |
| Exact motion numbers - curves, durations, press scale, springs | `xaml-design-polish` (XAML) / `web-interface-polish` (web) |
| Is each XAML detail correct - focus, virtualization, reduced motion, spacing scale | `userinterface-wiki-uno` |
| Building the Loading / Empty / Error states of a component | `uno-component-states` |
| Material color roles, type ramp, control style mechanics | `uno-material` |
| Dead code, perf, stack currency, build hygiene | `uno-audit` |
| Runtime proof that a change actually landed | `uno-verify` |
| **This skill** | Which improvements are warranted, in what order, to what standard, without disturbing the approved direction |

Invoke the owning skill when a pass reaches its territory, and cite it while writing per CLAUDE.md. On conflict about a mechanism, the owning skill wins.

## Step 1 - Diagnose before touching anything (mandatory)

Never apply all twelve passes by default. Applying every optimization to already-good design overworks it, and that is the primary failure mode of this skill.

Read or view the current work first, then produce a diagnosis:

```
## Diagnosis

Current direction (preserving):
- <palette / type / layout archetype / signature element, named>

Findings:
- <symptom> - <specific location, file:line or element name> - <pass>
- ...

Passes warranted: <list, in run order>
Passes skipped: <list> - <one-line reason each>
```

Rules:

- Every finding names a **specific location**. "Spacing feels off" is not a finding; "card padding is 12/16/20 across three sibling cards - `HomePage.xaml:44,61,78`" is.
- If fewer than three passes are warranted, say so and run only those. A tight design honestly needs Fit & Finish alone.
- If the diagnosis concludes the *direction itself* is the problem, stop and say so. That is an `xaml-art-direction` job and needs the user's approval, because it breaks the prime directive.

## Step 2 - Select passes

Twelve passes exist. Select from them.

| # | Pass | Goal | Detail |
|---|---|---|---|
| 1 | Refinement | Sharpen execution of what is already correct | [references/01-passes-structure.md](references/01-passes-structure.md) |
| 2 | Resolution | Turn unfinished or ambiguous decisions into deliberate ones | [references/01-passes-structure.md](references/01-passes-structure.md) |
| 3 | Coherence | Make repeated things behave like one system | [references/01-passes-structure.md](references/01-passes-structure.md) |
| 4 | Art Direction | Composed rather than merely arranged | [references/02-passes-composition.md](references/02-passes-composition.md) |
| 5 | Hierarchy | Information priority readable at a glance | [references/02-passes-composition.md](references/02-passes-composition.md) |
| 6 | Maturation | Prototype quality to shipped quality | [references/03-passes-production.md](references/03-passes-production.md) |
| 7 | Interaction | Behaves like a finished product across its states | [references/04-passes-craft.md](references/04-passes-craft.md) |
| 8 | Hardening | Survives real-world usage | [references/03-passes-production.md](references/03-passes-production.md) |
| 9 | Content Reality | Convincing when content stops being convenient | [references/03-passes-production.md](references/03-passes-production.md) |
| 10 | Fit & Finish | The final 10% of craftsmanship | [references/04-passes-craft.md](references/04-passes-craft.md) |
| 11 | Production Readiness | Could genuinely ship | [references/03-passes-production.md](references/03-passes-production.md) |
| 12 | Quality Bar | Anything still below standard | this file, Step 4 |

Read a pass's reference file before running it. Do not run a pass from memory of its title.

Honour scoped requests exactly:

- `Run a Gold Standard Pass on this screen` - full diagnostic, then the strongest warranted combination.
- `Do a Resolution + Art Direction Pass` - those two, plus the Quality Bar gate, and nothing else.
- `Keep the visuals as-is, run Hardening + Production Readiness` - the visual layer is frozen; report visual findings without acting on them.
- `Only Fit & Finish` - one pass. Resist expanding scope; list what you noticed elsewhere under Deliberately left alone.

Symptom-to-pass lookup: [references/05-diagnostic.md](references/05-diagnostic.md).

## Step 3 - Run in priority order

When several passes are warranted, run them in this order. The order exists so later passes refine earlier decisions instead of fighting them.

1. Resolve fundamental issues (Resolution)
2. Strengthen hierarchy and composition (Hierarchy, Art Direction)
3. Establish coherence (Coherence, Refinement)
4. Mature prototype-level decisions (Maturation)
5. Harden against real conditions (Hardening, Content Reality)
6. Improve interactions (Interaction)
7. Apply fit and finish (Fit & Finish)
8. Verify production readiness (Production Readiness)
9. Quality Bar gate (always last)

**No-undo rule.** A later pass may sharpen a decision an earlier pass made deliberately; it may not silently reverse one. Reversing requires stating the earlier decision, why it fails, and what replaces it. Coherence normalizing away a hierarchy contrast that Hierarchy just introduced is the classic self-inflicted regression.

**Hierarchy variation outranks mechanical uniformity.** Similar things behave similarly; different things differ on purpose.

## Step 4 - Quality Bar gate (always runs last)

Review the result as if it ships publicly tomorrow. Look for anything still accidental, generic, misaligned, inconsistent, unbalanced, unclear, over-dense, over-sparse, visually weak, unfinished, prototype-like, or over-designed.

Fix what materially improves the product. **Do not manufacture issues to justify further changes** - reporting "nothing below bar" is a valid and frequent outcome.

Then answer the final review, out loud, in the output:

- Is anything still obviously prototype-level?
- Does every major decision feel intentional?
- Is hierarchy immediately understandable?
- Does it feel coherent as a system?
- Does realistic content break anything?
- Are important states missing?
- Is anything visually distracting without adding value?
- Does it feel composed rather than assembled?
- Was the original direction preserved?
- Did every change materially improve the result?

For a running Uno app, "preserved" is proven at runtime, not by reading markup: verify the touched surfaces via `uno-verify` before claiming the gate passed.

## The decision rule

Before every change, ask:

> Does this improve clarity, usability, coherence, intentionality, resilience, or craftsmanship while preserving the established direction?

If not, leave it alone.

## Numbers, not adjectives

State every finding and every fix as a measurement. "Tighter spacing" is unbuildable; "card padding normalized from 12/16/20 to 16 across all three" is. Same for type sizes, radii, opacity, stroke weights, touch targets, and durations. Motion numbers come from the owning polish skill instead of being invented here.

## Moves that are not polish

These read as decoration applied to hide an unresolved decision. They are forbidden as a way to "add polish", in any pass:

- colored or highlighted left borders on containers
- glossy gradients, generic 3D, glass effects added for their own sake
- new cards, borders, shadows, or dividers introduced to create separation that spacing and hierarchy should carry
- decorative flourishes, TLDR boxes, robot/AI mascots
- swapping a distinctive choice for a design-system default because the default is safer
- reorganizing layout without a usability or compositional reason
- arbitrary color changes

Solve hierarchy with position, scale, weight, contrast, spacing, and grouping before reaching for anything drawn.

## Stop condition

Continue until further changes would give diminishing returns or start changing the approved direction. Then stop and say the work is at bar. Overrunning that line is how a preservation pass turns into an unrequested redesign.

## Output format

```
## Diagnosis
<per Step 1>

## Changes
<grouped by pass>
- <file:line> - <what changed, in numbers> - <reason>

## Deliberately left alone
- <thing> - <why it is already right, or why it is out of scope>

## Quality Bar
<final review answered; remaining items, or "at bar">

## Unresolved Questions
- ...
```

The **Deliberately left alone** section is mandatory. It is the evidence that the pass was diagnostic rather than exhaustive.

## Reference files

- [references/01-passes-structure.md](references/01-passes-structure.md) - Read before running Refinement, Resolution, or Coherence
- [references/02-passes-composition.md](references/02-passes-composition.md) - Read before running Art Direction or Hierarchy
- [references/03-passes-production.md](references/03-passes-production.md) - Read before running Maturation, Hardening, Content Reality, or Production Readiness
- [references/04-passes-craft.md](references/04-passes-craft.md) - Read before running Interaction or Fit & Finish
- [references/05-diagnostic.md](references/05-diagnostic.md) - Read during Step 1: symptom-to-pass lookup table and the preservation inventory

## Anti-pattern

Running all twelve passes on a design that needed two, and delivering something the user no longer recognizes. The tell is a change list with no reasons, no *Deliberately left alone* section, and a new palette or layout nobody asked for. The desired reaction is **"this feels finished"**, never **"this looks redesigned"**.

## When NOT to use

The user wants a redesign, a new visual identity, or a first draft of something that does not exist yet. Throwaway fixtures and repro projects. Code-quality-only work with no design surface - that is `uno-audit`.
