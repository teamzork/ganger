---
name: ganger-done
description: Mark a phase as fully merged and unblock downstream phases. Use when the user runs "ganger done <N>", "phase <N> is merged", "mark phase <N> complete", or after a PR has landed and the phase is fully merged into main.
---

Mark a phase as merged and surface any newly unblocked phases. Follow these steps exactly.

## Step 1 — Get the phase number

If the user provided a phase number (e.g. `/ganger:done 2`), use that. If not, fetch and display current state and ask: "Which phase has merged?"

## Step 2 — Fetch and validate

Run `git fetch origin`.
Read: `git show origin/main:.ganger/TEAM-STATE.md`

Find the row for the requested phase.

- If status is already `merged`: "Phase <N> is already marked as merged."
- If status is `available` or `in-progress`: Warn — "Phase <N> hasn't been marked for review yet. Are you sure it merged? (y/n)". Proceed if confirmed.
- If status is `review`: This is the expected state. Continue.

## Step 3 — Update TEAM-STATE.md on main

Stash current work, switch to main, pull:
```bash
git stash
git checkout main
git pull origin main
```

Edit `.ganger/TEAM-STATE.md`:

In the Slices table, find the phase row and update:
- Owner → `—`
- Status → `merged`
- Branch → `—`
- Updated → today's date

Commit and push:
```bash
git add .ganger/TEAM-STATE.md
git commit -m "ganger: phase <N> merged"
git push origin main
```

Handle push failure the same way as in `ganger-claim`.

Return to the previous branch:
```bash
git checkout -
git stash pop
```

## Step 4 — Check for unblocked phases

Read the `## Dependency Graph` section of TEAM-STATE.md.

Find all phases whose dependencies are now all `merged` after this update, AND whose current status is `available`.

These phases are newly claimable. Print:
```
✓ Phase <N> marked merged.
```

If any phases are newly unblocked:
```
  Now claimable:
    Phase <X>  <title>
    Phase <Y>  <title>

Run /ganger:claim <N> to take one.
```

If nothing is unblocked, just print the confirmation and nothing else.
