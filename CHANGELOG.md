# Changelog

All notable changes to Ganger are documented here. The `/ganger-update` command displays the latest entry after pulling.

---

## v0.1.7 — 2026-03-21

**Interactive numbered menus**

Every skill now ends with a colored numbered menu (`🟢  1`, `🔵  2`, `🟡  3`). Contributors pick a number instead of remembering commands. The agent waits for input and executes the chosen action. Menus are contextual — options adapt based on what you own and what's available.

---

## v0.1.6 — 2026-03-21

**Color-coded next steps**

Added consistent colored next-step blocks across all skills and the README. Green for primary actions, blue for secondary, yellow for informational.

---

## v0.1.5 — 2026-03-20

**`/ganger-update` command**

New skill to pull the latest Ganger release from GitHub and replace installed skill files. Auto-detects platform (Claude Code, Codex CLI, Antigravity) and updates Paul commands if installed.

---

## v0.1.4 — 2026-03-20

**Branch + project context on claim**

`/ganger-claim` now creates and pushes a feature branch with upstream tracking, and prints the full project status table plus any handoff notes — replacing the need for a separate `/ganger-status` call.

---

## v0.1.3 — 2026-03-19

**`/ganger-merge` command (direct merge, no PR)**

New skill for merging a phase branch directly into main without a pull request. Includes safety warnings, handoff note prompt, conflict handling, and downstream unblock detection.

---

## v0.1.2 — 2026-03-18

**Colon syntax + Paul support**

Skills renamed from dash to colon syntax (`ganger:claim` instead of `ganger-claim`). Added Paul framework support with flat `.md` command files in `~/.claude/commands/`.

---

## v0.1.1 — 2026-03-17

**Bug fixes from first test run**

Fixed stash/branch-order bugs in `ganger-done` and `ganger-handoff` — stash pop no longer runs unconditionally. Corrected skill install paths in README.

---

## v0.1.0 — 2026-03-16

**Initial release**

Six core skills: `init`, `setup`, `status`, `claim`, `handoff`, `done`. Spec and README. Git-based coordination via `.ganger/TEAM-STATE.md` — no server, no accounts.
