# Ganger

> Multi-agent team coordination for GSD projects. Claim a slice, run your agent, ship without colliding.

Ganger is a coordination layer on top of the [GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) framework that makes it work for teams. GSD is a powerful solo-developer workflow — fresh agent contexts, atomic commits, wave-based parallel execution. But it breaks the moment a second person shows up. Ganger fixes that.

Works with **Claude Code**, **Codex CLI**, and **Antigravity**. Team members can use different tools.

---

## The Problem

GSD assumes one human directing one agent. When two developers share a GSD project:

- Running `/gsd:execute-phase` from two machines overwrites each other's `.planning/` state
- There's no concept of who owns what phase — anyone can corrupt anyone's in-progress work
- No way to signal "I'm done, pick this up" with context attached
- No visibility into what your teammate's agent is currently building

---

## How It Works

Ganger partitions a GSD roadmap into **slices** — one phase per contributor, owned end-to-end. Each contributor claims a slice, works it on their own branch using normal GSD commands, and signals completion with a handoff note. The next contributor reads that note before touching a line of code.

The only shared state is a single markdown file — `.ganger/TEAM-STATE.md` — committed to the repo and updated via git. No server. No account. No new infrastructure.

```
  Phase 1  Auth & Roles           ✓ merged
  Phase 2  Doctor Profiles        ⚡ in-progress   @dan   (feat/ganger-phase-2-dan)
  Phase 3  Case Submission        ⚡ in-progress   @maya  (feat/ganger-phase-3-maya)
  Phase 4  Matching & Checkout    ○ available      depends on: 2, 3
  Phase 5  Review Dashboard       ○ available      depends on: 4
  Phase 6  Notifications          ○ available      depends on: 4, 5
```

---

## Core Principles

1. **Slice ownership, not free-for-all.** A claimed phase is locked. Others are warned before touching it.
2. **Agent-agnostic, Claude Code-first.** One skill package, three platforms.
3. **Git as the coordination medium.** No server. Markdown files on a shared branch.
4. **Async-first.** Handoff notes carry context forward so teammates don't need to be online at the same time.
5. **Minimal ceremony.** Claim a slice, run your agent, push when done.

---

## Commands

| Command | Who runs it | What it does |
|---|---|---|
| `/ganger:init` | Team lead, once | Reads GSD roadmap, creates TEAM-STATE.md, all phases set to available |
| `/ganger:setup` | Each contributor, once | Sets local identity handle, pulls current state |
| `/ganger:status` | Anyone, anytime | Pulls latest state and prints who owns what |
| `/ganger:claim <N>` | Contributor | Claims a phase, creates branch, warns on unmet dependencies |
| `/ganger:handoff` | Contributor | Marks phase ready for review, prompts for a context note |
| `/ganger:done <N>` | Anyone | Marks phase merged, unblocks downstream slices |

Codex CLI users prefix with `$ganger:` instead of `/ganger:`.

---

## Multi-Agent Support

Ganger uses the open [Agent Skills](https://agentskills.io) standard — the same `SKILL.md` format recognized by Claude Code, Codex CLI, and Antigravity. Each contributor installs the skill for their own tool. The coordination state is platform-agnostic.

| Platform | Skills path | Command prefix |
|---|---|---|
| Claude Code | `~/.claude/skills/ganger/` | `/ganger:` |
| Codex CLI | `~/.agents/skills/ganger/` | `$ganger:` |
| Antigravity | `~/.agent/skills/ganger/` | `/ganger:` |

One contributor can use Claude Code, another Codex CLI — they both read and write the same `TEAM-STATE.md` via git.

---

## Installation

Each contributor installs independently for their own agent:

```bash
git clone https://github.com/team-zork/ganger-skill ./ganger-tmp

# Install for your agent (pick one):
cp -r ganger-tmp/. ~/.claude/skills/ganger/     # Claude Code
cp -r ganger-tmp/. ~/.agents/skills/ganger/      # Codex CLI
cp -r ganger-tmp/. ~/.agent/skills/ganger/       # Antigravity
```

Requires: git, GSD installed, and any one of Claude Code / Codex CLI / Antigravity.

---

## Example — SecondMD (Teledoctor Marketplace)

Two developers, **@maya** (Claude Code) and **@dan** (Codex CLI), building a SaaS where patients pay specialists for second medical opinions.

**Day 0** — Maya runs GSD to generate the roadmap. Six phases: Auth, Doctor Profiles, Case Submission, Matching & Checkout, Review Dashboard, Notifications. She runs `/ganger:init`. Dan clones and runs `$ganger:setup`.

**Day 1** — Both check `/ganger:status`. Maya claims Phase 1 (Auth), Dan claims Phase 2 (Doctor Profiles). No overlap. Both run normal GSD workflows on their own branches simultaneously.

**Day 2** — Maya's auth implementation finishes first. She runs `/ganger:handoff` and leaves a note:

> Auth is Better Auth with three roles: patient, doctor, admin. `session.user.role` is set on the session object. Doctor accounts require `doctor.verified = true` before they can accept cases.

She opens a PR. Dan reviews, merges, runs `$ganger:done 1`, and immediately claims Phase 3 (Case Submission) while Maya stays on Phase 2. Two slices running in parallel — different tables, no conflict.

**Day 3** — Dan finishes Phase 2 and leaves a handoff note with the full doctor schema. Maya's agent reads it before writing a line of Phase 3 code. No ramp-up call. No Slack archaeology.

**Day 4 final state:**

```
  Phase 1  Auth & Roles           ✓ merged
  Phase 2  Doctor Profiles        ✓ merged
  Phase 3  Case Submission        ✓ merged
  Phase 4  Matching & Checkout    ⚡ in-progress   @dan
  Phase 5  Review Dashboard       ⚡ in-progress   @maya
  Phase 6  Notifications          ⏳ review        @dan   (PR open)
```

Full walkthrough in [docs/SPEC.md](docs/SPEC.md#example-workflow).

---

## Name

*Ganger* — English for the foreman of a work gang. Also the literal second half of *doppelgänger* (German: *gänger* = walker). The one who coordinates the gang, and the other that shadows you.

---

## Status

v0.1 alpha — in design. See [docs/SPEC.md](docs/SPEC.md) for the full specification.
