
Merge the current phase branch directly into main, bypassing a pull request. Follow these steps exactly.

## Step 1 — Read identity and current branch

Read `.ganger/config.md` to get the contributor's handle. If it doesn't exist, stop: "Run `/ganger:setup` first."

Get the current branch: `git branch --show-current`

If the current branch is `main` or not a `feat/ganger-phase-*` branch, stop:
"You're not on a Ganger phase branch. Switch to your feature branch first."

## Step 2 — Confirm the working tree is clean

Run `git status --porcelain`. If there are uncommitted changes, stop:

```
✗ You have uncommitted changes. Commit or stash them before merging.
```

Do not proceed with a dirty working tree — a merge to main should contain only intentional, committed work.

## Step 3 — Find the owned phase in TEAM-STATE.md

Fetch latest state: `git fetch origin`
Read: `git show origin/main:.ganger/TEAM-STATE.md`

Find the row where Branch matches the current branch AND Owner matches the local handle. Extract Phase number and Title.

If no match: "This branch is not registered in TEAM-STATE.md. Check with your team or run `/ganger:claim` first."

## Step 4 — Warn and confirm

Print this warning verbatim, then ask for confirmation:

```
⚠️  You are about to merge Phase <N> (<title>) directly into main.

This skips code review. Best practice: let your teammates know before
doing this — a quick message goes a long way. Teammates can't catch
issues they never saw.

Merge directly without a PR? (y/n)
```

If the user answers anything other than `y` or `yes`, stop: "Merge cancelled. Consider running `/ganger:handoff` to open a normal PR."

## Step 5 — Prompt for a handoff note

Ask:
```
Leave a note for your teammates (describe what you built, any schema decisions, API shapes, or things they should know):
>
```

Read the user's input. Optional — if they skip it, remind them:
"No note added. Your teammates will need to read the code to understand what landed."

## Step 6 — Merge into main

Switch to main and pull latest:
```bash
git checkout main
git pull origin main
```

Merge the feature branch with `--no-ff` to preserve branch history in the log:
```bash
git merge --no-ff <feature-branch> -m "ganger: merge phase <N> (<title>) [direct, no PR]"
```

If the merge has conflicts, stop and print:
```
✗ Merge conflict detected. Resolve conflicts, then run this command again.

Tip: fix the conflicts on your feature branch first:
  git checkout <feature-branch>
  git merge main
  # resolve conflicts, commit
  git checkout main
  git merge --no-ff <feature-branch>
```

Do not auto-resolve conflicts. The contributor must resolve them manually.

## Step 7 — Update TEAM-STATE.md

Edit `.ganger/TEAM-STATE.md`:

1. In the Slices table, find the phase row and update:
   - Owner → `—`
   - Status → `merged`
   - Branch → `—`
   - Updated → today's date

2. In the `## Notes` section, append (if a note was provided):
   ```
   > **Phase <N> — <title>** (@<handle>, <date>) ⚡ direct merge
   > <note text>
   ```
   If no note was provided, still append the header line with `⚡ direct merge` so teammates can see this phase landed without a PR.

Commit everything together and push:
```bash
git add .ganger/TEAM-STATE.md
git commit -m "ganger: phase <N> merged (direct, no PR) — <handle>"
git push origin main
```

If the push is rejected (someone else pushed to main while you were working):
```bash
git pull --rebase origin main
git push origin main
```

If the rebase also fails, stop and ask the user to resolve manually.

## Step 8 — Surface newly unblocked phases

Read the `## Dependency Graph` section of TEAM-STATE.md.

Find all phases whose dependencies are now all `merged` AND whose current status is `available`. These are newly claimable.

Print:
```
✓ Phase <N> merged directly into main.

  Reminder: let your teammates know what landed — they can't catch
  issues they never saw.
```

If any phases are newly unblocked:
```
  Now claimable:
    Phase <X>  <title>
    Phase <Y>  <title>

Run /ganger:claim <N> to take one.
```
