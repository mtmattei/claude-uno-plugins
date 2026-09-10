---
name: demo-ready
description: Preflight an app before a demo, presentation, or screen recording and produce a walkthrough beat script. Use whenever the user is about to demo, present, record, or show an app, asks if it's "ready", or mentions a demo deadline - by default for any Uno Platform app, page, or sample headed for an audience or camera.
---

# Demo Ready

Flush everything that can go wrong live, then script the walkthrough. The user rambles under pressure (see `user-presentation-style` memory): the beat script is part of the deliverable, not an extra.

## Inputs

- Project dir; minutes until demo (affects depth); recording? (yes → record-first rule applies).

## Steps

1. **Clean build** of the demo TFM. Flag warnings, missing dependencies, config issues.
2. **Launch and exercise everything** via the uno-app MCP (use `uno-verify` mechanics): every screen, every nav path, every interactive state, empty/error states, window resize. Note startup time and first-frame visuals.
3. **Risk list:** anything slow, flaky, network-dependent, or one click away from an error dialog. For each: avoid it, fix it, or give a fallback line.
4. **Beat script** - a table, one row per beat:

| Say | Do | Point at |
|---|---|---|
| One spoken sentence | One action | One thing on screen |

   Rules baked in: say the line, then do the click - never talk while typing. One-sentence spine per section as the lost-my-place recovery line. Verbatim segues between sections. Fallback line for every risky beat ("never debug live").
5. **If recording:** the FIRST action of the actual run is starting capture (ffmpeg gdigrab recipe in `reference-screen-record-ffmpeg` memory; prepend the WinGet Links dir to PATH). Confirm "recording is live" before anything else. Don't take screenshots, don't ask for permissions mid-recording, wait for the user's go-ahead before navigating.

## Output

- Verdict first: 🟢 ready / 🟡 ready with cautions (name them) / 🔴 blockers (name them).
- Issue list with fixes applied or recommended.
- The beat script.

## Time budget

"Demo in 30 minutes" means: build + smoke every screen + top-3 risks + short beat script. Skip the audit-grade sweep. Never spend the runway polishing.

## Example

> "I'm about to demo this project and need to make sure it is fully ready beforehand"
→ build, launch, walk all screens via MCP, report 🟡 (search is slow on first query - fallback line provided), deliver 8-beat script.

## Anti-pattern

Scaffolding, fixing, and configuring off-camera and starting the recording afterwards. If recording the process is the point, every un-recorded step is lost work (see `feedback-record-first`).
