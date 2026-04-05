---
name: linear-project
model: sonnet
description: Linear project-level operations — board overview and project scaffolding. Use when the user wants to "show the board", "project overview", "project status", "what's blocked", "what's next", "scaffold a project", "set up a project", "plan a project board", "create milestones", "scaffold a milestone", "scope a milestone", "scaffold issues for a milestone", or needs a high-level view of project progress. For individual issue operations (create, update, comment, handoff), see the linear-issue skill instead. For methodology and process, see the dev-cycle skill.
---

# Linear Project

Project-level operations for Linear. Board overview for situational awareness, and scaffolding to go from a plan to a structured board.

For individual issue operations (create, update, comment, handoff), use the `linear-issue` skill.
For methodology and operating model (ticket anatomy, dev cycle, progressive scoping), see the `dev-cycle` skill.

## Setup

Before first use in a session, load Linear MCP tools:

```
ToolSearch: "+linear save_project"
ToolSearch: "+linear get_project"
ToolSearch: "+linear list_projects"
ToolSearch: "+linear save_milestone"
ToolSearch: "+linear list_milestones"
ToolSearch: "+linear list_cycles"
ToolSearch: "+linear save_issue"
ToolSearch: "+linear get_issue"
ToolSearch: "+linear list_issues"
ToolSearch: "+linear list_issue_labels"
ToolSearch: "+linear list_issue_statuses"
ToolSearch: "+linear list_teams"
```

The ToolSearch queries above will resolve to the correct Linear MCP tools regardless of naming convention. Use whatever tool names ToolSearch returns.

## Intent Detection

Parse `$ARGUMENTS` to determine mode.

| Signal | Mode | Example |
|---|---|---|
| Empty | **Ask** | `/linear-project` |
| "board"/"overview"/"status"/"dashboard" + project | **Board** | `/linear-project board polymarket-sim` |
| Project name alone | **Board** | `/linear-project polymarket-sim` |
| "scaffold"/"setup"/"plan"/"create project" + project name | **Scaffold Milestones** | `/linear-project scaffold polymarket-sim` |
| "scaffold"/"scope" + project/milestone path | **Scaffold Issues** | `/linear-project scaffold polymarket-sim/API Foundation` |

Project name with no verb defaults to **Board** (safe, read-only).

## Shared Context

### Labels

These are the workspace labels — use them when creating issues during scaffolding:

- **Bug** — broken things, regressions, errors
- **Feature** — new functionality, improvements to existing features
- **Release** — shipping, publishing, deploying
- **Test** — testing work, QA, coverage
- **Scope** — exploration, research, spikes, planning
- **User** — requires human input or action outside of code

Every issue should get at least one label. When the work type is ambiguous: refactoring/cleanup → Feature, research/planning → Scope, needs human action → User.

### Statuses

| Type | Status | Shorthand |
|---|---|---|
| backlog | Backlog | `backlog` |
| unstarted | Todo | `todo` |
| started | In Progress | `progress` / `wip` / `start` |
| started | In Review | `review` |
| completed | Done | `done` / `close` / `complete` |
| canceled | Canceled | `canceled` / `cancel` |
| canceled | Duplicate | `duplicate` / `dupe` |

When setting status via `save_issue`, use the `state` field (e.g., `state: "In Review"`).

## Board Mode

Project-level overview. Shows all issues grouped by status with progress metrics — the way to understand the state of a large project at a glance.

1. **Resolve project** — Call `get_project` with `includeMilestones: true`
2. **Fetch all issues** — Call `list_issues` filtered by project (paginate if >50, up to 250 per call)
3. **Group by status** and output:

```
## Board: polymarket-sim
Status: Active | Lead: kylesnav
Progress: 12/30 issues done (40%)

### Milestones
- M1: API Foundation — In-Dev (5/8 done, target: 2026-04-01)
- M2: UI Layer — Pre-Dev (0/10 done, target: 2026-04-15)
- M3: Integration — Pre-Dev (0/6 done, target: 2026-05-01)

### By Status

**In Review (2)**
  KS-31: Add retry logic to API calls
  KS-32: Implement caching layer

**In Progress (4)**
  KS-33: Build dashboard component (kylesnav)
  KS-34: Add WebSocket support (kylesnav)
  KS-35: Refactor auth middleware (unassigned)
  KS-36: Write migration scripts (unassigned)

**Todo (8)**
  KS-37: Add rate limiting ← blocked by KS-33
  KS-38: Implement search endpoint
  KS-39: Add export functionality
  [+5 more]

**Backlog (4)** [collapsed]

**Done (12)** KS-10, KS-11, KS-12 [+9 more]

### Blocked
Issues with unresolved blockedBy relations:
- KS-37: Add rate limiting ← waiting on KS-33 (In Progress)
- KS-40: Deploy to staging ← waiting on KS-31 (In Review)

### Next Up
Highest-priority unassigned Todo issues:
1. KS-38: Implement search endpoint (High)
2. KS-39: Add export functionality (Normal)
```

Key behaviors:
- Active work (In Review, In Progress) shows first — that's what needs attention now
- Done and Backlog are collapsed — not actionable
- "Blocked" highlights issues whose `blockedBy` targets are not yet Done
- "Next Up" shows highest-priority unassigned Todo items — what an agent or person should pick up next
- Omit Milestones section if the project has none
- To detect blocked issues: check each issue's relations for `blockedBy` where the blocking issue's status is not Done

### Milestone Phase Indicators

Each milestone shows its dev-cycle phase based on issue status distribution:

- **Pre-Dev**: All issues are Backlog or Todo (scoping and requirements phase)
- **In-Dev**: At least one issue is In Progress or In Review (implementation underway)
- **Post-Dev**: All issues are Done, Canceled, or Duplicate (knowledge work phase)

This gives a quick read on where each milestone sits in the development lifecycle without needing to open individual issues.

## Scaffold Mode

Two levels of scaffolding. Each level creates structure at exactly one depth — never deeper. This enforces progressive scoping: you build the skeleton first, and details emerge when you start working.

### Level 1: Scaffold Milestones

Triggered by: "scaffold `<project>`" or "set up milestones for `<project>`"

Creates milestones for a project. Does NOT create issues — that's Level 2.

1. **Gather the plan** — Look for a **Milestone Roadmap** in the project's docs folder (e.g., `docs/Milestone Roadmap.md`). This is the Claude Code-facing roadmap produced during Project Initialization (see `dev-cycle` skill). It contains scope, acceptance criteria, and scope boundaries per milestone — exactly what you need to create well-structured milestones. If no Milestone Roadmap exists, tell the user to create one first — don't scaffold from vague descriptions. The user may also provide:
   - A PM Roadmap (the product strategy version — useful for context but the Milestone Roadmap is the source of truth for scaffolding)
   - Additional context or adjustments on top of the roadmap

2. **Decompose into milestones** — Each milestone represents a dev cycle:
   - Clear deliverable or outcome
   - Pre-Dev / In-Dev / Post-Dev phases (from the dev-cycle methodology)
   - Target date (if known)
   - 2-5 milestones is typical, but follow the project's natural structure

3. **Present for approval**:

```
## Scaffold: polymarket-sim

### Project
Name: polymarket-sim
Description: Prediction market simulation platform
Lead: kylesnav

### Milestones
| # | Name | Target | Outcome |
|---|---|---|---|
| 1 | API Foundation | 2026-04-15 | Core API endpoints and data models operational |
| 2 | UI Layer | 2026-05-01 | Dashboard and market views rendering real data |
| 3 | Integration & Launch | 2026-05-15 | End-to-end flow working, deployed to production |

Create? (y/n)
```

4. **Wait for approval** — Use AskUserQuestion
5. **Create** — Project (if new) via `save_project` with `name`, `description`, `lead: "me"`, `addTeams: ["kylesnav"]`. Then milestones via `save_milestone` with `project`, `name`, and `targetDate`.
6. **Report** — Project URL and milestone summary

### Level 2: Scaffold Issues within a Milestone

Triggered by: "scaffold issues for `<milestone>`", "scope `<milestone>`", or "scaffold `<project>`/`<milestone>`"

Creates issues for a specific milestone. Issues are flat — no parent-child nesting at scaffold time. Dependencies are expressed via `blockedBy` relations.

1. **Gather context** — The primary source is the milestone's section in the **Milestone Roadmap** (`docs/Milestone Roadmap.md`). Each milestone section contains scope, what to build, technical references, open decisions, acceptance criteria, and scope boundaries. Use these to decompose into issues. Also consider:
   - The previous milestone's Post-Dev findings (if any) — these feed forward and may adjust scope
   - Additional specs or requirements the user provides on top of the roadmap
   - If the Milestone Roadmap doesn't exist or doesn't cover this milestone, ask the user for requirements rather than guessing

2. **Decompose into issues** — Each issue is a standalone unit of work:
   - Scoped to 1 issue = 1 PR (if the work would need multiple PRs, split it)
   - Gets a Standard tier description (ticket anatomy — see `linear-issue` skill)
   - Gets appropriate label(s)
   - Gets `blockedBy` relations where real ordering constraints exist
   - Dependencies should reflect actual technical constraints, not just preferred sequencing

3. **Present for approval**:

```
## Scaffold: API Foundation (in polymarket-sim)

### Issues (6 total)
| # | Title | Labels | Dependencies |
|---|---|---|---|
| 1 | Design API schema and data models | Scope | — |
| 2 | Implement market CRUD endpoints | Feature | blocked by #1 |
| 3 | Add authentication middleware | Feature | — |
| 4 | Build data ingestion pipeline | Feature | blocked by #1 |
| 5 | Write API integration tests | Test | blocked by #2, #3 |
| 6 | Set up CI/CD for API service | Release | blocked by #5 |

Create all? (y/n)
```

4. **Wait for approval** — Use AskUserQuestion
5. **Create in dependency order** — Blockers first so you have their IDs for `blockedBy` fields. Each issue gets a Standard tier description (ticket anatomy with completion checklist). For large scaffolds (6+ issues), offer the option to create with Quick descriptions and flesh them out individually later.
6. **Report** — List all created issues with identifiers and URLs

### What scaffold mode does NOT do

- **Never scaffolds sub-issues.** Sub-issues emerge during implementation as the engineer discovers what's needed to complete a parent issue. If asked to scaffold sub-issues, explain: "Sub-issues are created during development as implementation details surface. Scaffold the parent issues now — sub-issues will be created when work begins on each issue."
- **Never creates both levels at once.** Scaffold milestones OR scaffold issues within a milestone — not both in one operation. If the user wants the full structure, do Level 1 first, get approval, then do Level 2 for each milestone separately.

### Scaffolding Guidelines

- Every issue gets at least one label
- Issues start as **Todo** (ready to be picked up) unless they're blocked by an issue that hasn't been created yet, in which case **Backlog**
- Keep issue titles short and imperative
- Scope issues to 1 PR each — if a task would need multiple PRs, split it
- Dependencies reflect real ordering constraints, not just preferred sequencing
- Don't over-decompose — if a task takes less than an hour, it's probably a sub-issue that should emerge during execution, not a standalone scaffolded issue
- Issue descriptions use the Standard tier (ticket anatomy) — each issue should be an agent-executable spec with context, requirements, acceptance criteria, and a completion checklist

## Examples

### Board overview
```
/linear-project board polymarket-sim
```
Shows project progress, milestone status with phase indicators, issues grouped by status, blocked items, and next available tasks.

### Board by project name only
```
/linear-project polymarket-sim
```
Same as above — project name alone defaults to Board.

### Scaffold milestones for a project
```
/linear-project scaffold polymarket-sim
```
Gathers plan details, presents milestone structure with target dates and outcomes, creates project and milestones after approval.

### Scaffold issues for a milestone
```
/linear-project scaffold polymarket-sim/API Foundation
```
Gathers requirements for the milestone, presents flat issue list with labels and dependencies, creates issues with Standard tier descriptions after approval.

### Bare invocation
```
/linear-project
```
Asks what the user wants to do.

## Error Handling

| Error | Response |
|---|---|
| Project not found | Show available projects from `list_projects` |
| Milestone not found | Show available milestones for the project from `list_milestones` |
| No issues in project | "polymarket-sim has no issues yet. Want to scaffold them?" |
| Milestone creation fails | Show error, continue with remaining milestones |

## Related Skills

| Skill | Relationship |
|---|---|
| `dev-cycle` | Methodology and operating model — milestone lifecycle, progressive scoping |
| `linear-issue` | Individual issue operations — create, update, comment, handoff |
| `using-git-worktrees` | Create isolated worktree when starting an issue |
| `finishing-a-development-branch` | PR creation/merge when completing an issue |
