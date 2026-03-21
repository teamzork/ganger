---
name: ganger:setup
description: Set up Ganger for a contributor on a cloned repo. Use when the user runs "ganger setup", "set up my ganger identity", or has just cloned a repo that already has Ganger initialized and needs to register themselves.
---

Set up Ganger for this contributor. Follow these steps exactly.

## Step 1 — Check preconditions

- Confirm `.ganger/TEAM-STATE.md` exists. If not, tell the user: "Ganger is not initialized on this repo. The team lead should run `/ganger-init` first."
- Check if `.ganger/config.md` already exists. If it does, read it and ask: "You're already set up as `<handle>`. Re-configure? (y/n)". If no, stop.

## Step 2 — Get the contributor's handle

Ask:
> "What's your handle? (e.g. @maya)"

Ensure it starts with `@`. If the user provides a name without `@`, prepend it automatically.

## Step 3 — Write config

Create `.ganger/config.md` with this content:

```markdown
# Ganger Local Config
handle: <handle>
```

## Step 4 — Confirm gitignore

Check that `.ganger/config.md` is in `.gitignore`. If not, append it. This file must never be committed — it is per-contributor identity.

## Step 5 — Pull and show current state

Run:
```
git fetch origin
git show origin/main:.ganger/TEAM-STATE.md
```

Parse and display the current slice status using the same format as `/ganger-status`:

```
Ganger — Project Status

  Phase 1  <title>   ○ available
  Phase 2  <title>   ○ available
  ...

✓ Set up as <handle>. You're ready to contribute.

What would you like to do?
  🟢  1 → Claim a phase
  🟡  2 → View status in detail
```

## Step 6 — Wait and execute

Wait for the user to type a number. Then execute:
- `1` → Ask "Which phase?" then run the `/ganger-claim` flow.
- `2` → Run the `/ganger-status` flow.

If the user types anything other than a valid number, say: "Pick a number from the menu, or type what you'd like to do."
