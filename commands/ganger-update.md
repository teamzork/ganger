
Pull the latest Ganger release from GitHub and replace the installed skill files. Follow these steps exactly.

## Step 1 — Detect the current platform

Check which install path exists to determine the platform:

1. Check `~/.claude/skills/ganger-claim/SKILL.md` → **Claude Code**
2. Check `~/.agents/skills/ganger-claim/SKILL.md` → **Codex CLI**
3. Check `~/.agent/skills/ganger-claim/SKILL.md` → **Antigravity**

If none are found, stop and say: "Ganger doesn't appear to be installed. See the README for installation instructions: https://github.com/teamzork/ganger"

Record the skills directory (e.g. `~/.claude/skills`).

Also check if Paul commands are installed at `~/.claude/commands/ganger-init.md`. If so, note that commands need updating too.

## Step 2 — Clone the latest version

Clone into a temporary directory:

```bash
git clone --depth 1 https://github.com/teamzork/ganger /tmp/ganger-update
```

If the clone fails, stop and say: "Could not reach https://github.com/teamzork/ganger — check your network connection."

## Step 3 — Copy updated skills

Remove the old skill directories and copy the new ones:

```bash
rm -rf <skills-dir>/ganger-*
cp -r /tmp/ganger-update/skills/ganger-* <skills-dir>/
```

Where `<skills-dir>` is the path detected in Step 1.

If Paul commands were detected in Step 1, also update them:

```bash
rm -f ~/.claude/commands/ganger-*.md
cp /tmp/ganger-update/commands/ganger-*.md ~/.claude/commands/
```

## Step 4 — Clean up

```bash
rm -rf /tmp/ganger-update
```

## Step 5 — Print confirmation

```
✓ Ganger updated to latest version.

Updated skills in: <skills-dir>
```

If Paul commands were also updated, add:
```
Updated commands in: ~/.claude/commands/
```

End with:
```
Restart your agent session to pick up the new skills.
```
