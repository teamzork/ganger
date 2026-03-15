---
name: ganger-init
description: Initialize Ganger team coordination on a project. Use when the user runs "ganger init", "set up ganger", "initialize team state", or wants to start using Ganger on a repo for the first time.
---

Initialize Ganger team coordination for this project. Follow these steps exactly.

## Step 1 — Check preconditions

- Run `git rev-parse --is-inside-work-tree` to confirm this is a git repo. If not, stop and tell the user.
- Check if `.ganger/TEAM-STATE.md` already exists. If it does, stop and say: "Ganger is already initialized. Run `/ganger-status` to see current state."
- Check that there is at least one remote configured (`git remote -v`). Warn (but do not block) if no remote exists — TEAM-STATE.md updates require a shared remote to be useful.

## Step 2 — Gather phases

First, check for known roadmap files in this order:
1. `.planning/ROADMAP.md` (GSD)
2. `ROADMAP.md` (root)
3. `docs/ROADMAP.md`

If a roadmap file is found, read it and extract any phase or milestone names listed. Ask the user:
> "Found a roadmap at `<path>`. Import phases from it? (y/n)"

If yes, parse the phases from that file. Look for numbered items, headings with "Phase", or milestone entries. Extract the title of each phase.

If no roadmap file exists, or the user declines the import, prompt:
> "List your phases, one per line. Add `| depends on: 2,3` after a title to set dependencies. Leave a blank line when done."

Read the user's input line by line until a blank line is received.

## Step 3 — Confirm the phase list

Show the user the phases you have collected in a numbered list and ask: "Does this look right? (y/n)"

If no, go back to Step 2.

## Step 4 — Create Ganger files

Create the `.ganger/` directory.

Create `.ganger/TEAM-STATE.md` with this exact structure, substituting the phases gathered above:

```markdown
# Ganger — Team State
_Initialized: <today's date>_

## Slices

| Phase | Title | Owner | Status | Branch | Updated |
|---|---|---|---|---|---|
| 1 | <title> | — | available | — | — |
| 2 | <title> | — | available | — | — |

## Dependency Graph

<list each dependency as: "Phase N depends on: Phase X, Phase Y">
<if no dependencies, write: "No dependencies defined.">

## Notes

_Handoff notes will appear here as phases complete._
```

For phases with no dependencies, omit them from the Dependency Graph section.

## Step 5 — Update .gitignore

Check if `.gitignore` exists. If it does, check whether `.ganger/config.md` is already listed. If not, append this line:
```
.ganger/config.md
```

If no `.gitignore` exists, create one with that single line.

## Step 6 — Commit

Stage and commit only `.ganger/TEAM-STATE.md` and `.gitignore`:
```
git add .ganger/TEAM-STATE.md .gitignore
git commit -m "chore: init ganger team state"
```

## Step 7 — Print onboarding instructions

Print this message, substituting the actual remote URL if available:

```
✓ Ganger initialized with <N> phases.

Each teammate should:
  1. Clone or pull this repo
  2. Run /ganger-setup  (or $ganger-setup on Codex CLI)
  3. Run /ganger-status to see available slices
  4. Run /ganger-claim <N> to take a phase

Push this commit so teammates can pull the initial state:
  git push origin main
```
