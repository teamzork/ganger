---
name: ganger:status
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

If this fails (no remote, or file doesn't exist on main), fall back to reading the local `.ganger/TEAM-STATE.md`. If that also doesn't exist, stop and say: "Ganger is not initialized. Run `/ganger-init` to get started."

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

What would you like to do?
  🟢  1 → Claim a phase
  🟡  2 → View a phase in detail
```

Build this menu dynamically based on context:
- Always show option `1 → Claim a phase` if any phases are `available` or unblocked.
- If the contributor owns an in-progress phase (see Step 4), show `🔵  1 → Hand off my phase` as the first option instead, and shift "Claim" to option 2.
- If any phases are in `review` status, add: `🟡  N → Mark a phase done`
- Never show more than 3 options.

If all phases are merged, print: "✓ All phases complete." and skip the menu.

## Step 4 — Read local handle and adjust menu

If `.ganger/config.md` exists, read the local handle. If the current contributor owns any in-progress slices, show:
```
You own: Phase <N> (<branch>)
```
And reorder the menu so option 1 is their most likely next action (handoff).

## Step 5 — Wait and execute

Wait for the user to type a number. Then execute the corresponding command:
- "Claim a phase" → ask "Which phase?" then run the `/ganger-claim` flow
- "Hand off my phase" → run `/ganger-handoff`
- "Mark a phase done" → ask "Which phase?" then run `/ganger-done`
- "View a phase in detail" → ask "Which phase?" then show its notes and status

If the user types anything other than a valid number, say: "Pick a number from the menu, or type what you'd like to do."
