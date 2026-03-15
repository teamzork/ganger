# Ganger — v0.1 Alpha Specification

**Goal:** Prove the core coordination loop works for a two-person team on a shared GSD project. No new infrastructure. No agent intelligence beyond what GSD already provides. Just ownership, locking, and status — in markdown files, via git.

Alpha is complete when two developers can work the same GSD project simultaneously without corrupting each other's planning state.

---

## Scope

**In:** slice claiming, branch creation, TEAM-STATE.md, conflict warnings, handoff signal, status command, skill package for Claude Code / Codex CLI / Antigravity.

**Out:** rate limit budgeting, file-scope conflict detection, MCP server, real-time visibility, automated dependency blocking.

---

## File Structure

Ganger adds one directory to the repo:

```
.ganger/
├── TEAM-STATE.md       # Shared coordination state — committed to main
└── config.md           # Local identity config — gitignored
```

### `.ganger/TEAM-STATE.md`

```markdown
# Ganger — Team State
_Last updated: 2026-03-15 by @maya_

## Slices

| Phase | Title | Owner | Status | Branch | Updated |
|---|---|---|---|---|---|
| 1 | Auth & Sessions | @maya | merged | — | 2026-03-10 |
| 2 | Upload Pipeline | @dan | in-progress | feat/ganger-phase-2-dan | 2026-03-14 |
| 3 | Image Transform | — | available | — | — |
| 4 | Public Gallery | — | available | — | — |

## Dependency Graph

- Phase 3 depends on: Phase 2
- Phase 4 depends on: Phase 3

## Notes

> @dan: Phase 2 upload endpoint is at `/api/upload` — uses presigned R2 URLs. Phase 3 can assume that contract.
```

### `.ganger/config.md`

```markdown
# Ganger Local Config
handle: @dan
```

Gitignored. Set once on first run. Identifies who is operating the local instance.

---

## Commands

Six slash commands. All are thin orchestrators — they read/write markdown and run git, nothing more.

---

### `/ganger:init`

Run once per project by the team lead after GSD's roadmap exists.

**What it does:**
1. Reads `.planning/ROADMAP.md` and extracts all phases.
2. Creates `.ganger/TEAM-STATE.md` with every phase set to `available`.
3. Adds `.ganger/config.md` to `.gitignore`.
4. Commits TEAM-STATE.md to the current branch with message `chore: init ganger team state`.
5. Prints instructions for teammates: clone, run `/ganger:setup`, start claiming.

**Does not:** create branches, touch planning files, or require teammates to be present.

---

### `/ganger:setup`

Run once per contributor after cloning.

**What it does:**
1. Prompts for the contributor's handle (e.g. `@maya`).
2. Writes `.ganger/config.md` with that handle.
3. Confirms the gitignore entry exists.
4. Pulls latest TEAM-STATE.md and prints current slice status.

---

### `/ganger:status`

Print the current state of all slices. Can be run any time.

**Output format:**

```
Ganger — Project Status (pulled 2 minutes ago)

  Phase 1  Auth & Sessions       ✓ merged
  Phase 2  Upload Pipeline       ⚡ in-progress   @dan  (feat/ganger-phase-2-dan)
  Phase 3  Image Transform       ○ available      depends on: Phase 2
  Phase 4  Public Gallery        ○ available      depends on: Phase 3

Run /ganger:claim <phase> to take a slice.
```

Pulls latest TEAM-STATE.md from remote before rendering. Never reads stale local state.

---

### `/ganger:claim <phase>`

Claim an available slice and begin work.

**What it does:**
1. Pulls latest TEAM-STATE.md. Errors if the slice is not `available`.
2. Checks if the slice's dependencies are `merged`. If not, prints a warning:
   ```
   ⚠ Phase 3 depends on Phase 2, which is still in-progress (@dan).
   Claim anyway? (y/n)
   ```
   Allows override — does not hard block in alpha.
3. Creates branch: `feat/ganger-phase-<N>-<handle>` from current main.
4. Updates TEAM-STATE.md: sets owner, status → `in-progress`, branch name, timestamp.
5. Commits the TEAM-STATE.md update directly to main via a short-lived branch + immediate merge (or pushes to main if permissions allow). This is the only shared mutation.
6. Checks out the contributor's new feature branch.
7. Prints: `Slice claimed. You're on feat/ganger-phase-3-maya. Run your GSD phase commands as normal.`

From this point the contributor works exactly as they would in solo GSD — discuss, plan, execute, verify — on their own branch.

---

### `/ganger:handoff`

Signal that a slice is complete and ready for the next person.

**What it does:**
1. Confirms the current branch matches the claimed slice branch.
2. Prompts for an optional note (e.g. "API contract is at `/api/upload`, presigned R2 URLs").
3. Updates TEAM-STATE.md: status → `review`, appends note to the Notes section.
4. Commits TEAM-STATE.md update to main (same short-lived merge pattern as claim).
5. Prints the PR creation command with the branch pre-filled.

Does not open the PR automatically — the contributor does that. Ganger just flips the status and surfaces the note for teammates.

---

### `/ganger:done <phase>`

Mark a slice fully merged. Run by any contributor after the PR lands.

**What it does:**
1. Updates TEAM-STATE.md: status → `merged`, clears owner and branch.
2. Checks if any `available` slices were blocked on this phase and surfaces them as newly claimable.
3. Commits to main.
4. Prints any slices now unblocked: `Phase 3 is now claimable.`

---

## TEAM-STATE.md Update Protocol

The only tricky part of the alpha: TEAM-STATE.md lives on `main` but contributors work on feature branches. Updates must land on main without a full PR review cycle.

**Alpha approach — direct commit to main:**
- Ganger creates a minimal commit touching only TEAM-STATE.md.
- Pushes directly to main (or to a `ganger/state` branch that is auto-merged by convention).
- If the team has branch protection on main, they add a bypass rule for the TEAM-STATE.md path, or use the `ganger/state` branch pattern.
- Merge conflicts on TEAM-STATE.md are resolved by last-write-wins for the `Status` column — the file is append-only for Notes.

This is the known rough edge of v0.1. A later version replaces it with an MCP server or a Supabase row.

---

## Multi-Agent Compatibility

A team does not have to use the same tool. One contributor can use Claude Code, another Codex CLI, another Antigravity — all coordinating through the same `.ganger/TEAM-STATE.md`. The coordination layer is platform-agnostic; only the skill installation differs.

### The Agent Skills Standard

Claude Code, Codex CLI, and Antigravity all share the open Agent Skills standard — the same `SKILL.md` format is recognized by all three. Ganger ships one canonical skill package that installs into each platform's expected path.

| Platform | Skills path | Instructions file | Command prefix |
|---|---|---|---|
| Claude Code | `.claude/skills/ganger/` | `CLAUDE.md` | `/ganger:` |
| Codex CLI | `.codex/skills/ganger/` or `~/.agents/skills/ganger/` | `AGENTS.md` | `$ganger:` |
| Antigravity | `.agent/skills/ganger/` | `.agent/rules/ganger.md` | `/ganger:` |

The `.ganger/` directory and `TEAM-STATE.md` are shared and identical across all three. No platform-specific state.

### What Each Contributor Does

Each contributor installs Ganger for their own agent. From that point they run the same conceptual commands — the prefix differs, the behavior is identical.

```bash
# Claude Code contributor
/ganger:claim 3

# Codex CLI contributor
$ganger:claim 3

# Antigravity contributor
/ganger:claim 3
```

All three update the same `TEAM-STATE.md` via the same git protocol.

### Tool Name Adapter

The SKILL.md uses neutral language ("read the file", "commit this change") and each platform's agent interprets that natively. No adapter scripts needed in v0.1 — the operations are simple enough that natural language instructions are sufficient.

If a future version needs to invoke shell helpers directly, it will ship platform-specific script wrappers following GSD Multi's adapter pattern (separate `openai.yaml` per platform in the skill package).

### Platform Notes

**Codex CLI** — Uses `$ganger:` prefix. The SKILL.md `description` field handles implicit activation: when a user describes claiming a slice or checking team status, Codex auto-loads the Ganger skill without explicit invocation. Global install at `~/.agents/skills/ganger/` makes it available across all projects.

**Antigravity** — Google's agent-first IDE (VS Code fork, Nov 2025). Supports Claude Sonnet 4.6 and Opus 4.6 as selectable models. GSD has already been ported to Antigravity by the community, meaning contributors who prefer it are already in the GSD mental model.

---

## Example Workflow — SecondMD (Teledoctor Second Opinion Marketplace)

A two-person team building a SaaS product where patients upload medical records and pay specialists for a second opinion. **@maya** uses Claude Code. **@dan** uses Codex CLI.

### The Project

**SecondMD** — patients submit a case (medical records, scans, description), browse verified specialists by condition and price, pay for a review, and receive a written second opinion within 48 hours.

Roles: `patient`, `doctor`, `admin`. Core flow: case submission → specialist matching → payment → review → delivery.

### Day 0 — Project Setup

Maya initializes the GSD project, completes research and roadmap, then brings Dan in. The roadmap lands with 6 phases:

```
Phase 1 — Auth & Roles          (patient / doctor / admin, Better Auth)
Phase 2 — Doctor Profiles       (onboarding, specialties, pricing, availability)
Phase 3 — Case Submission       (record upload to R2, condition tagging, brief)
Phase 4 — Matching & Checkout   (browse specialists, Stripe payment, case assignment)
Phase 5 — Review Dashboard      (doctor queue, response editor, delivery)
Phase 6 — Notifications         (email on case received, review ready, Resend)
```

Maya runs `/ganger:init`. Dan clones the repo and runs `$ganger:setup`, entering `@dan` as his handle.

### Day 1 — Both Start at Once

Both contributors run `/ganger:status`, see all phases available, and claim independently:

```
/ganger:claim 1     →  feat/ganger-phase-1-maya created, checked out
$ganger:claim 2     →  feat/ganger-phase-2-dan created, checked out
```

TEAM-STATE.md after both claims:

```markdown
| 1 | Auth & Roles        | @maya | in-progress | feat/ganger-phase-1-maya | 2026-03-15 |
| 2 | Doctor Profiles     | @dan  | in-progress | feat/ganger-phase-2-dan  | 2026-03-15 |
| 3 | Case Submission     | —     | available   | —                        | —          |
| 4 | Matching & Checkout | —     | available   | —       depends on: 2, 3 | —          |
```

Each runs their normal GSD workflow on their branch. `.planning/` artifacts stay on respective branches — no collision.

### Day 2 — Maya Finishes First

Auth is simpler than doctor onboarding. Maya's agent finishes Phase 1 and runs:

```
/ganger:handoff
```

Note left:
> Auth is Better Auth with three roles: patient, doctor, admin. Role is set on the session object as `session.user.role`. Middleware at `middleware.ts` checks role before routing to `/dashboard`, `/doctor`, `/admin`. Doctor accounts require `doctor.verified = true` before they can accept cases.

Phase 1 → `review`. Maya opens her PR. Dan reviews, merges, runs `$ganger:done 1`.

Maya immediately claims Phase 3 (Case Submission) — no dependency issues. Two slices now run in parallel: Phase 2 touches `doctors` schema, Phase 3 touches `cases` schema and R2 upload. Different files, different tables.

### Day 3 — Dan Finishes, Handoff to Checkout

Dan's agent wraps Phase 2 and runs:

```
$ganger:handoff
```

Note left:
> Doctor profiles live in `doctors` table (id, user_id, bio, verified, hourly_rate, response_hours). Specialties are many-to-many via `doctor_specialties`. Availability is a JSON array of weekday windows in `doctors.availability`. Presigned R2 upload for avatars hits `POST /api/doctor/avatar`. Unverified doctors cannot appear in search results.

Phase 4 is still blocked on Phase 3. Dan claims Phase 6 (Notifications) speculatively — Resend integration and email templates don't touch anything Phase 5 owns:

```
$ganger:claim 6
```
```
⚠ Phase 6 depends on Phase 5 (Review Dashboard), which is available (unclaimed).
  Claim anyway? (y/n): y
Slice claimed. Your branch will need to merge Phase 5 before notification triggers exist.
```

### Day 4 — Final Convergence

Maya finishes Phase 3 and hands off with the full case schema. Dan reads both prior handoff notes (Phase 2 + Phase 3) before writing a line of Phase 4. Zero ramp-up.

```
$ganger:done 3
$ganger:claim 4     →  feat/ganger-phase-4-dan
/ganger:claim 5     →  feat/ganger-phase-5-maya (override, stubs assignment logic)
```

Final state:

```
  Phase 1  Auth & Roles           ✓ merged
  Phase 2  Doctor Profiles        ✓ merged
  Phase 3  Case Submission        ✓ merged
  Phase 4  Matching & Checkout    ⚡ in-progress   @dan
  Phase 5  Review Dashboard       ⚡ in-progress   @maya
  Phase 6  Notifications          ⏳ review        @dan   (PR open)
```

### Why It Works Here

- **Phases 1 and 2 in parallel:** Auth schema and doctor schema touch different tables.
- **Phases 2 and 3 in parallel:** `doctors` and `cases` are separate; both reference `specialties` but Phase 3 reads it only.
- **Phase 4 claimed before Phase 3 merges:** Ganger warns but allows. Handoff note covers the gap.
- **Phase 6 claimed speculatively:** Resend setup and email templates are independent of Phase 5's code. Rebase when Phase 5 merges.
- **Handoff notes replace sync calls:** Neither contributor had to explain their schema live. The note is there, the agent reads it, work starts.

---

## What v0.1 Does Not Do

- No automatic file-scope conflict detection (manual discipline required)
- No rate limit budgeting across team members
- No real-time state (TEAM-STATE.md is pull-on-demand only)
- No MCP server
- No automated PR creation or merging
- No notification system (Slack, email, etc.)
- No UI
- No GitHub Copilot CLI support (three platforms is enough for alpha)

---

## Success Criteria

1. Two developers can run `/ganger:claim` on different phases without corrupting each other's planning state.
2. `/ganger:status` always reflects current team state after a pull.
3. A completed slice surfaces a note visible to the next contributor before they start.
4. The full GSD workflow (discuss → plan → execute → verify) runs unmodified inside a claimed branch.
5. No special server setup required — works on any repo with git remote access.
6. A Claude Code user and a Codex CLI user can coordinate on the same project using the same TEAM-STATE.md.

---

## Open Questions

- Should slice ownership live in git (markdown files) or in a lightweight hosted service (Supabase/Convex) for real-time visibility?
- How does this interact with GSD v2's unique milestone IDs and programmatic git model?
- Is the right unit of ownership a *phase* or a *plan within a phase*? Finer granularity enables more parallelism but adds more coordination overhead.
- How do you handle a slice that turns out to need touching files already owned by another in-progress slice? Escalation path?
- Can the agent itself detect when it's about to write a file outside its claimed scope and self-correct?

---

## Prior Art

- **GSD v2** — adds unique milestone IDs to prevent naming collisions across branches; closest existing work
- **GSD Multi** — multi-platform support (Claude Code, Copilot CLI, Codex CLI) sharing the same planning state; inspiration for the agent-agnostic adapter pattern
- **GitHub Projects + Copilot Workspace** — team task assignment but no structured agent context management
- **Linear + Claude** — issue tracking with AI assist, but no execution coordination layer
