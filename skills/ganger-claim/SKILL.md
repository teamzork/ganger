---
name: ganger:claim
description: Claim a phase and begin work. Use when the user runs "ganger claim <N>", "claim phase <N>", "I want to work on phase <N>", or asks to take ownership of a slice.
---

Claim a phase slice and set up the contributor's branch. Follow these steps exactly.

## Step 1 — Read identity

Read `.ganger/config.md` to get the contributor's handle. If the file doesn't exist, stop and say: "Run `/ganger-setup` first to register your handle."

## Step 2 — Get the phase number

If the user provided a phase number in their message (e.g. `/ganger-claim 3`), use that. If not, run `/ganger-status` output and ask: "Which phase would you like to claim?"

## Step 3 — Fetch and validate

Run `git fetch origin` to get the latest state.

Read TEAM-STATE.md from the remote: `git show origin/main:.ganger/TEAM-STATE.md`

Find the row for the requested phase. Check:

**If status is not `available`:** Stop and report:
- `in-progress` → "Phase <N> is already claimed by <owner> on `<branch>`."
- `review` → "Phase <N> is in review by <owner>. Wait for it to merge or run `/ganger-done <N>` after the PR lands."
- `merged` → "Phase <N> is already merged."

**If status is `available`:** Continue.

## Step 4 — Check dependencies

Read the `## Dependency Graph` section. Find any dependencies for this phase.

For each dependency, check its status in the Slices table:
- If all dependencies are `merged`: proceed silently.
- If any dependency is `in-progress` or `review`: print a warning:
  ```
  ⚠ Phase <N> depends on Phase <X> (<title>), which is <status> (<owner>).
  Claim anyway? (y/n)
  ```
  Wait for the user's response. If `n`, stop. If `y`, continue with a note that they may need to rebase later.

## Step 5 — Update TEAM-STATE.md on main first

This is the coordination-critical step. Update the shared state on main *before* creating the feature branch. This avoids any need to stash — main is the starting point.

```bash
git checkout main
git pull origin main
```

Edit `.ganger/TEAM-STATE.md`: in the Slices table, find the row for this phase and update:
- Owner → `<handle>`
- Status → `in-progress`
- Branch → `<branch-name>` (use the name you will create: `feat/ganger-phase-<N>-<handle-without-@>`)
- Updated → today's date (YYYY-MM-DD)

Then commit and push:
```bash
git add .ganger/TEAM-STATE.md
git commit -m "ganger: claim phase <N> (<handle>)"
git push origin main
```

If the push fails due to branch protection, print:
```
⚠ Could not push directly to main (branch protection enabled).
  Options:
  1. Ask your team lead to add a bypass rule for .ganger/TEAM-STATE.md
  2. Use the ganger/state branch pattern: push to origin/ganger/state and merge manually
  For now, the local TEAM-STATE.md has been updated but is not yet shared.
```

## Step 6 — Create and check out the feature branch

Now create the feature branch from the updated main:

```bash
git checkout -b feat/ganger-phase-<N>-<handle-without-@>
```

If the branch already exists locally, stop and say: "Branch `<branch>` already exists locally. Did you mean to continue existing work?"

No stash needed — the branch is created fresh from main after the state update.

## Step 7 — Print confirmation

```
✓ Phase <N> claimed — <title>
  Branch: feat/ganger-phase-<N>-<handle>

You're now on your feature branch. Work on this phase as normal.
When you're done, run /ganger-handoff to signal completion.
```
