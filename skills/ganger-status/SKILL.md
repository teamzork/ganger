---
name: ganger-status
description: Show current Ganger team state — who owns what phase, what's in progress, what's available. Use when the user runs "ganger status", "what's the team working on", "show team state", "what phases are available", or wants to check coordination state before claiming work.
---

Display the current Ganger team state. Always fetch from remote first — never read stale local state.

## Step 1 — Fetch latest state

Run:
```
git fetch origin 2>/dev/null
```

Then read the TEAM-STATE.md directly from the remote main branch (not the local working copy, which may be on a feature branch):
```
git show origin/main:.ganger/TEAM-STATE.md
```

If this fails (no remote, or file doesn't exist on main), fall back to reading the local `.ganger/TEAM-STATE.md`. If that also doesn't exist, stop and say: "Ganger is not initialized. Run `/ganger:init` to get started."

Note the fetch timestamp so you can show "pulled N minutes ago" in the header.

## Step 2 — Parse the Slices table

Read the `## Slices` table from TEAM-STATE.md. Extract each row: Phase number, Title, Owner, Status, Branch.

Also read the `## Dependency Graph` section to know which phases are blocked.

## Step 3 — Render the status display

Print a formatted status display. Use these symbols:
- `✓` for `merged`
- `⚡` for `in-progress`
- `⏳` for `review`
- `○` for `available`
- `⊘` for `available` but blocked by an unmerged dependency

Format each line as:
```
  Phase <N>  <title padded to 22 chars>  <symbol> <status>   <owner if set>  (<branch if set>)
```

If a phase is blocked, append: `depends on: Phase <N>`

Example output:
```
Ganger — Project Status  (fetched just now)

  Phase 1  Auth & Roles            ✓ merged
  Phase 2  Doctor Profiles         ⚡ in-progress   @dan   (feat/ganger-phase-2-dan)
  Phase 3  Case Submission         ⚡ in-progress   @maya  (feat/ganger-phase-3-maya)
  Phase 4  Matching & Checkout     ⊘ blocked        depends on: Phase 2, Phase 3
  Phase 5  Review Dashboard        ○ available      depends on: Phase 4
  Phase 6  Notifications           ○ available      depends on: Phase 4, Phase 5

Run /ganger:claim <N> to take a slice.
```

If all phases are merged, print: "✓ All phases complete."

## Step 4 — Read local handle (optional)

If `.ganger/config.md` exists, read the local handle. If the current contributor owns any in-progress slices, append a note:
```
You own: Phase <N> (<branch>)
```
