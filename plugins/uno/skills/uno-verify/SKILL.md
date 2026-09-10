---
name: uno-verify
description: Build, launch, and runtime-verify an Uno Platform app in one pass. Use whenever asked to build, run, launch, relaunch, "fire it up", test, or confirm a change works in any Uno Platform app, or after any code change that needs runtime confirmation - by default, even when the user only says "build and run".
---

# Uno Verify

One command for the build → launch → verify loop. Verify runtime behavior yourself via the uno-app MCP. Never ask the user to check something the MCP can answer.

## Inputs

- Project dir (default: cwd). TFM (default: `net10.0-desktop`).
- Optional: reference images for visual parity (default threshold ≥85%).

## Steps

1. **Locate the `.sln`/`.slnx`.** The session and the uno-app MCP must run in the folder that directly contains it. Confirm `.mcp.json` sits beside it; if missing, seed from `~/.claude/templates/uno/.mcp.json`.
2. **Build** the desktop TFM with `dotnet build`. Surface errors verbatim. Zero warnings is the bar on Release.
3. **Launch via `uno_app_start`** in Debug, devserver connected, hot reload live. Never bare `dotnet run` when the uno-app MCP is available. Announce when the window is ready.
4. **Verify with the MCP, not the user:** `uno_app_visualtree_snapshot` for structure, `uno_app_element_peer_default_action` for interactions (pointer_click coordinates miss; peer refs don't). Exercise the states the change touched.
5. **If reference images were given:** compare, list concrete numeric deltas (radius, spacing, color), fix, re-verify. Iterate to the parity threshold.
6. **Leave the app running** between iterations. Do not close/relaunch unless the change requires a restart (XAML hot reload covers most visual edits).

## Goal-based invocation

When the request is fix-until-verified (not a one-shot check), run the loop goal-based instead of turn-based:

1. **State the goal and stop condition up front**, quantitatively, before the first edit. Good exit conditions: build passes with 0 warnings, visual tree contains element X with property Y=Z, reference parity ≥85%.
2. **Iterate autonomously** against MCP verification (steps above) until the exit condition is met — no check-ins between attempts.
3. **Hard cap: 4 attempts.** Two failed cycles on the *same root cause* still triggers the Debug Checkpoint early (see Failure protocol). Hitting the cap without convergence also ends in a Debug Checkpoint, never a fifth blind attempt.

The native `/goal` command is the same shape ("/goal: <exit condition>, stop after 4 tries") — suggest it when the user is typing the loop manually.

## Failure protocol

- uno-app tools missing or `MCP error -32001`: confirm cwd contains the sln → `/mcp` reconnect once (warm retry usually lands; cold DevServer spawns can exceed 30s; `MCP_TIMEOUT=120000` is already set). Fallback: global tool `uno-devserver --mcp-app`.
- Hot reload not propagating: check the app was launched Debug + devserver-connected (step 3); relaunch correctly rather than patching blind.
- Two failed fix cycles on the same root cause: stop and run the Debug Checkpoint format from CLAUDE.md. Do not grind.

## Output

Short status: built (warnings count) / launched / what was verified and how / anything off. No screenshot requests to the user.

## Example

> "make the chip radius 16 then uno-verify"
→ edit, build, hot-reload or relaunch, snapshot the chip element, confirm CornerRadius=16 in the visual tree, report.

## Anti-pattern

Launching with `dotnet run`, then asking "can you check if the page renders now?" - that is two violations: wrong launcher (no devserver/hot reload) and delegating verification to the user.
