---
name: dev-cycle
description: EPD development methodology and operating model. Use when starting work on a Linear issue, creating or scoping a ticket, beginning a milestone, discussing the dev cycle, or deciding how to structure work. Triggers on "start working on", "scope this ticket", "begin milestone", "dev cycle", "EPD", "ticket template", "completion checklist", "how do we work", "progressive scoping", "pre-dev", "post-dev", "what's the process", or any context where the user is creating, executing, or reviewing a Linear issue. Also use when the user references milestones, worktrees, or the relationship between Cowork and Claude Code.
---

# Dev Cycle

The operating model for shipping software using Cowork and Claude Code. This skill is methodology — it tells you WHY and HOW work is structured. For the mechanics of talking to Linear, see `linear-issue` and `linear-project`.

## EPD Roles

Two actors, two environments, complementary responsibilities:

- **Product + Design** — the user, working in Cowork. Owns direction, scoping, requirements, acceptance criteria, design review, knowledge work.
- **Engineering** — Claude, working in Claude Code. Owns implementation, testing, worktree management, PRs.

Both actors can invoke any skill. A Cowork session creating tickets and a Claude Code session executing them use the same vocabulary and templates — the ticket anatomy is the shared contract between product and engineering.

## Project Initialization

Before milestones exist, the project needs two roadmap artifacts. These are produced in Cowork conversation — product decisions, scope tradeoffs, and architecture calls that can't be automated. Neither skill generates these; they're the output of the user and Claude thinking through the project together.

### PM Roadmap

Lives in the project's documentation folder (e.g., `Documentation/MVP/Roadmap.md`). This is the product strategy document — it tells the story of why the project is structured this way.

Structure per milestone:
- **Pre-Dev** — What's being built, why it matters, what goal it serves
- **In-Dev** — Technical specs, key decisions, acceptance criteria
- **Post-Dev** — Specific knowledge work that feeds forward into the next milestone (design critique, architecture review, product decisions)

Also includes a **Setup** phase (before M1) and a **Release** phase (after the last milestone). Post-Dev phases explicitly name the Cowork skills that apply (e.g., `design:design-critique`, `engineering:architecture`).

### Milestone Roadmap (Claude Code-facing)

Lives in the project's docs folder (e.g., `docs/Milestone Roadmap.md`). This is the engineering contract — it tells Claude Code exactly what to build and what not to build.

Structure per milestone:
- **Scope** — One-line deliverable
- **What to build** — Concrete implementation items
- **Technical references** — Which docs to read and what to pull from them
- **Open decisions to resolve** — TBD items the engineer must resolve during this milestone
- **Acceptance criteria** — Pass/fail gates
- **Scope boundaries** — Explicit "do not build" list (prevents scope creep across milestones)

The Milestone Roadmap omits Pre-Dev and Post-Dev content — those are Cowork concerns, not engineering concerns. It also omits Setup and Release phases since those are PM-managed.

### Why two documents

The PM Roadmap captures the full EPD lifecycle including knowledge work between milestones. The Milestone Roadmap is a strict build spec. Combining them creates noise — the engineer doesn't need to know about post-dev design critiques, and the PM doesn't need acceptance criteria cluttering the strategic view. Keeping them separate means each document serves its audience cleanly.

### When to create them

Create both during the first Pre-Dev phase — before scaffolding milestones in Linear. The Milestone Roadmap is the input that `linear-project` scaffold mode consumes. If you're scaffolding milestones and there's no Milestone Roadmap, create it first.

## Milestone Lifecycle

Every milestone follows three phases. Each phase has a primary actor, but either can contribute.

### Pre-Dev

Primary actor: Cowork (product/design).

This is where direction gets set. Scoping, research, requirements gathering, ticket creation. Issues move from Backlog to Todo as they become ready for engineering.

Output: well-formed tickets with specs, acceptance criteria, and a completion checklist. Each ticket is an agent-executable spec — Claude Code in plan mode should be able to read a ticket and know exactly what to build, what patterns to follow, and how to verify the work.

### In-Dev

Primary actor: Claude Code (engineering).

Implementation happens here. Issues move from Todo → In Progress → In Review. Each issue gets its own git worktree (via `using-git-worktrees`) and produces exactly one PR (via `finishing-a-development-branch`).

Sub-issues emerge during this phase as the engineer discovers implementation details. They are not planned upfront — this is progressive scoping (see below).

### Post-Dev

Primary actor: Cowork (product/design).

Knowledge work that feeds forward into the next milestone's Pre-Dev. This phase is milestone-specific, not generic — each milestone produces different insights that shape what comes next.

Examples of what Post-Dev might include:
- Testing the build against real-world conditions and documenting findings
- Design critique comparing the implementation to the original intent
- Architecture review identifying patterns to carry forward or change
- Product decisions about scope adjustments for the next milestone
- Updating documentation, decision records, or project context

The key principle: Post-Dev is where product decisions, design critique, and architecture review happen. It's what makes this EPD rather than just "build, build, build." Every milestone's Post-Dev should be specific about what knowledge work happens — not a generic "review" step.

### Phase Detection

You can infer which phase a milestone is in from its issue statuses:
- **Pre-Dev**: All issues are Backlog or Todo
- **In-Dev**: At least one issue is In Progress or In Review
- **Post-Dev**: All issues are Done, Canceled, or Duplicate

## Ticket Anatomy

This is the template for what a well-formed issue looks like. Every substantive issue (features, meaningful bugs, scoping tasks) follows this structure. Trivial items (typos, config changes) can use a one-sentence description instead.

```markdown
## Context
Why this work exists. What problem it solves. What's insufficient about
the current state. Give the engineer enough background to make good
judgment calls during implementation.

## Requirements
What must be true when this is done. Concrete, testable statements.
These are the specification — they define the work.

## Acceptance Criteria
- [ ] Specific, pass/fail criterion
- [ ] Specific, pass/fail criterion
- [ ] Specific, pass/fail criterion

## Technical Approach
Written for an agent in plan mode. File paths to start from, patterns
to follow, key decisions already made. Skip this section if the
approach is obvious from the requirements.

## Completion Checklist
- [ ] Implementation complete
- [ ] Tests passing
- [ ] /simplify — review changed code for quality and efficiency
- [ ] /review — code review passes
- [ ] PR created (via finishing-a-development-branch)
```

### Why each section matters

**Context** gives the engineer judgment. Without it, they follow instructions literally and miss the intent. A good Context section means the engineer can make smart tradeoffs when edge cases arise.

**Requirements** are the contract. They're what the engineer builds to, and what acceptance criteria verify. They should be concrete enough to test but not so prescriptive that they dictate implementation.

**Acceptance Criteria** are the pass/fail gates. Each one is a checkbox because it's binary — either the criterion is met or it isn't. These get checked off during the review phase.

**Technical Approach** is optional but valuable for complex work. It reduces wasted exploration by pointing the engineer at the right files, patterns, and prior art. Written for plan mode means: specific enough that Claude can read it and start building without asking clarifying questions.

**Completion Checklist** is standard on every substantive issue. The `/simplify` and `/review` items mean actually running those skills — checking them off confirms the work was done, not just acknowledged. The checklist can be extended with issue-specific items (e.g., "Database migration tested locally"), but the base items are always present.

### Omit empty sections

An empty section is worse than no section. If the approach is obvious, skip Technical Approach. If there's only one acceptance criterion, that's fine. The template is a guide, not a form to fill in completely.

## 1 Issue = 1 Worktree = 1 PR

This is the fundamental unit-of-work constraint:

- **1 Issue** — a single Linear ticket with specs and acceptance criteria
- **1 Worktree** — an isolated git worktree created via `using-git-worktrees`
- **1 PR** — exactly one pull request produced from that worktree

If an issue would need multiple PRs, it's scoped too broadly — split it into multiple issues. If an issue is too small to justify a PR, it should probably be a sub-issue of something larger.

This constraint keeps work atomic and reviewable. Each PR maps to a clear intent (the issue), and each issue has a clear artifact (the PR).

## Progressive Scoping

Sub-issues are not planned upfront. They emerge during In-Dev as the engineer discovers the concrete steps needed to implement a parent issue.

**During Pre-Dev (scaffolding):** Create flat issues within a milestone. No sub-issues, no parent-child nesting. Dependencies between issues are expressed via `blockedBy` relations, not hierarchy.

**During In-Dev (execution):** As the engineer works a parent issue, implementation details surface. "Add authentication layer" might spawn sub-issues like "Set up JWT middleware" and "Add token refresh endpoint." These get created via `linear-issue`'s sub-issue mode.

**Why this works:** Upfront decomposition is speculative — you're guessing at implementation details before you've touched the code. Progressive scoping defers decomposition until you have real context, which produces better-scoped sub-issues and avoids the overhead of pre-planning work that changes once you start building.

Sub-issues use brief descriptions (one to three sentences) and don't get the full completion checklist — they're implementation details of the parent, not standalone specs.

## Cross-References

These skills work together but independently — each functions standalone without the others loaded.

| Skill | When to use |
|---|---|
| `linear-issue` | Creating, reading, updating, commenting on individual Linear issues |
| `linear-project` | Board overview, scaffolding milestones and issues within milestones. Scaffold mode expects a Milestone Roadmap as input. |
| `using-git-worktrees` | Creating an isolated worktree when starting work on an issue |
| `finishing-a-development-branch` | Creating a PR or merging when completing an issue |
| `/review` | Code review — dispatches the code-reviewer subagent |
| `/simplify` | Reviews changed code for reuse, quality, and efficiency |
