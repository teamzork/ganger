---
name: ganger-handoff
description: Signal that a phase is complete and ready for review. Use when the user runs "ganger handoff", "I'm done with my phase", "mark phase ready for review", or wants to hand off their slice to the team.
---

Mark the current phase as complete and ready for review. Prompt for a handoff note. Follow these steps exactly.

## Step 1 — Read identity and current branch

Read `.ganger/config.md` to get the contributor's handle. If it doesn't exist, stop: "Run `/ganger:setup` first."

Get the current branch: `git branch --show-current`

## Step 2 — Find the owned phase

Fetch latest state: `git fetch origin`
Read: `git show origin/main:.ganger/TEAM-STATE.md`

Find the row where Branch matches the current branch AND Owner matches the local handle.

If no match is found:
- If the current branch is not a `feat/ganger-phase-*` branch: "You don't appear to be on a Ganger phase branch. Switch to your feature branch first."
- If the branch matches but the owner doesn't: "This branch is registered to a different owner in TEAM-STATE.md. Check with your team."

Extract the Phase number and Title from the matched row.

## Step 3 — Prompt for a handoff note

Ask:
```
Leave a note for the next contributor (recommended — describe contracts, schema decisions, API shapes):
>
```

Read the user's input. This can be multi-line — keep reading until the user signals done (blank line or explicit confirmation).

The note is optional. If the user provides nothing, proceed without a note and remind them:
"No note added. Consider leaving one — it saves the next contributor from reading your code."

## Step 4 — Update TEAM-STATE.md on main

Stash, switch to main, pull:
```bash
git stash
git checkout main
git pull origin main
```

Edit `.ganger/TEAM-STATE.md`:

1. In the Slices table, find the phase row and update:
   - Status → `review`
   - Updated → today's date

2. In the `## Notes` section, append the note (if provided) in this format:
   ```
   > **Phase <N> — <title>** (@<handle>, <date>)
   > <note text>
   ```
   If no note was provided, skip this.

Commit and push:
```bash
git add .ganger/TEAM-STATE.md
git commit -m "ganger: phase <N> ready for review (<handle>)"
git push origin main
```

Handle push failure the same way as in `ganger-claim` (branch protection warning).

Return to the feature branch:
```bash
git checkout <feature-branch>
```

Only run `git stash pop` if the earlier `git stash` output was NOT "No local changes to save". Do not unconditionally pop — it will fail with exit code 1 if nothing was stashed.

## Step 5 — Print PR instructions

```
✓ Phase <N> marked for review.

Open a pull request for your branch:
  gh pr create --base main --head <branch> --title "Phase <N>: <title>"

After the PR merges, any contributor can run:
  /ganger:done <N>
to unblock downstream phases.
```

If `gh` CLI is not available, show the GitHub URL pattern instead.
