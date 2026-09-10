# Universal Principles (Platform-Neutral)

These are the platform-neutral foundations that the original `userinterface-wiki` skill carried before it was retired. The XAML-specific rules in this folder assume them. They apply to any surface (XAML, web, native).

## Animation Principles

- **Animate with purpose.** Every animation answers "what changed and where did it come from?" Decorative motion with no informational job is a candidate for deletion.
- **Duration bands:** micro-interactions (hover, press, toggle) 100–200ms; state/component transitions 200–300ms; larger spatial moves (page, panel, connected element) 300–500ms. Beyond 500ms motion reads as lag unless it is deliberately theatrical.
- **Enter ≠ exit.** Entrances are slower and more expressive; exits are faster and subtler. Users care about what arrives; what leaves should get out of the way.
- **Origin matters.** Elements enter from where they conceptually live (a flyout scales from its trigger; a sheet rises from its edge). Modals are the exception — they stay centered.
- **Interruptibility.** Any animation tied to user input must be interruptible and reversible mid-flight. A press the user cancels should not finish playing the full press choreography.
- **Nothing appears from nothing.** Scale from ~0.95 + fade, never from 0. Real objects don't materialize from a point.

## Timing Functions

- **Ease-out for anything entering or responding to input** — instant start reads as responsiveness.
- **Ease-in-out for elements moving within the view** (position A to position B).
- **Plain ease-in is almost always wrong** on its own — it feels sluggish at the start, acceptable only as the exit half of a pair.
- **Linear only for continuous mechanical motion** (spinners, progress, marquees).
- **Never ship the platform default curve for expressive motion.** Defaults exist to be unnoticeable; house motion needs a chosen curve. (XAML house curves: see `xaml-design-polish`, which is authoritative for feel values.)

## Laws of UX (the ones that shape UI decisions)

- **Fitts's Law** — time to acquire a target scales with distance/size. Bigger, closer targets; generous hit areas (see `state-xaml-hit-target.md`).
- **Hick's Law** — decision time grows with the number of choices. Trim menus; progressive disclosure over walls of options.
- **Jakob's Law** — users expect your app to work like the apps they already use. Platform conventions beat clever novelty for core flows.
- **Miller's Law** — working memory holds ~7±2 items. Chunk lists, group settings, paginate.
- **Doherty Threshold** — keep response under ~400ms to hold attention; past that, show progress and preserve responsiveness (prefetch, skeletons — see `prefetch-xaml-*`).
- **Aesthetic–Usability Effect** — polished-looking UI is perceived as easier to use; polish buys forgiveness for minor friction, and that is a reason to invest in it, not a license to skip fixing the friction.
- **Peak–End Rule** — experiences are judged by their most intense moment and their ending. Spend polish on the hero interaction and the completion state.
