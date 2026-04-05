---
name: linear-issue
model: sonnet
description: Full Linear issue lifecycle. Use when the user wants to "create a Linear issue", "file a ticket", "read an issue", "show issue details", "update status", "close an issue", "mark done", "add a comment", "leave a handoff note", "check off items", "create a sub-issue", "scope a ticket", "write acceptance criteria", "ticket anatomy", or any mention of tracking/managing individual work items in Linear. Handles read, create, batch create, update, comment, checklist, sub-issue, and handoff workflows. For project-level operations (board overview, scaffolding), see the linear-project skill instead. For methodology and process, see the dev-cycle skill.
---

# Linear Issue

Lifecycle management for individual Linear issues. Read, create, update, comment, check off items, create sub-issues, and hand off between agents.

For project-level operations (board overview, project scaffolding), use the `linear-project` skill.
For methodology and operating model (ticket anatomy, dev cycle, progressive scoping), see the `dev-cycle` skill.

## Setup

Before first use in a session, load Linear MCP tools:

```
ToolSearch: "+linear save_issue"
ToolSearch: "+linear get_issue"
ToolSearch: "+linear list_issues"
ToolSearch: "+linear save_comment"
ToolSearch: "+linear list_comments"
ToolSearch: "+linear list_issue_statuses"
ToolSearch: "+linear list_issue_labels"
ToolSearch: "+linear list_projects"
ToolSearch: "+linear list_teams"
ToolSearch: "+linear list_milestones"
```

The ToolSearch queries above will resolve to the correct Linear MCP tools regardless of naming convention. Use whatever tool names ToolSearch returns.

## Intent Detection

Parse `$ARGUMENTS` to determine mode. **First match wins.**

| Signal | Mode | Example |
|---|---|---|
| Empty | **Ask** | `/linear-issue` |
| Identifier alone, or + "read"/"show"/"get" | **Read** | `/linear-issue KS-42` |
| Identifier + "status:"/"done"/"close"/"review" | **Update** | `/linear-issue KS-42 status:done` |
| Identifier + "comment"/"note"/"handoff" + text | **Comment** | `/linear-issue comment KS-43 "Watch for X"` |
| Identifier + "check"/"uncheck"/"checklist" | **Checklist** | `/linear-issue check KS-42 2` |
| "sub"/"child" + identifier + title | **Sub-issue** | `/linear-issue sub KS-42 Fix token skip` |
| "handoff" + identifier + "->" + identifier + text | **Compound Handoff** | `/linear-issue handoff KS-42 -> KS-43 "Watch for X"` |
| No identifier + descriptive text | **Create** | `/linear-issue Add health check endpoint` |
| Multiple items (list/commas) | **Batch Create** | `/linear-issue\n- Fix X\n- Add Y` |

Identifier with no verb defaults to **Read** (safe, non-destructive).

### When `$ARGUMENTS` is empty

Ask what they want to do. Don't guess.

## Defaults

| Field | Default | Override |
|---|---|---|
| Team | Auto-detect from Linear | User specifies a team |
| Assignee | `me` (self-assign) | User says "unassigned" or names someone |
| Priority | None | Infer from context unless user specifies urgency |
| Project | None | Infer from context or ask if ambiguous |
| Labels | Infer from context | User specifies |
| Milestone | None | Infer from project context or user specifies |

### Label inference

Every issue should get at least one label. These labels categorize the type of work and make boards scannable at a glance. Apply the best-fit label based on the input:

- **Bug** — "bug", "broken", "fix", "regression", "doesn't work", "error", "crash"
- **Feature** — "add", "new", "implement", "create", "build", "introduce"
- **Release** — "release", "publish", "submit", "ship", "launch", "distribute", "deploy"
- **Test** — "test", "write tests", "QA", "coverage", "e2e", "unit test", "verify", "integration test"
- **Scope** — "scope", "explore", "investigate", "spike", "research", "flesh out", "define", "evaluate", "assess"
- **User** — "needs human input", "manual", "user testing", "hands-on", "requires human", "review needed"

When the input doesn't match any keyword but describes work, use your judgment:
- Refactoring, optimization, cleanup, improving existing functionality → **Feature**
- Research, decision-making, architecture, planning → **Scope**
- Anything requiring a human to act outside of code → **User**

Multiple labels are fine when appropriate (e.g., Feature + Test for "build X and write tests for it").

### Project inference

If the user mentions a repo name or project name, check `list_projects` and assign if there's a clear match. If no match or ambiguous, skip — don't ask unless the user seems to expect project assignment.

### Milestone inference

If the issue belongs to a project with milestones (check via `list_milestones`), assign to the most relevant active milestone based on context. If the project has no milestones or the fit is unclear, skip.

## Scoping Principles

These principles shape how issues are created and decomposed:

- **1 issue = 1 PR**: Each substantive issue should be scoped to produce exactly one pull request. If the work would need multiple PRs, split into separate issues.
- **Agent-executable specs**: Write descriptions so that an agent in plan mode can read the ticket and know what to build, what patterns to follow, and how to verify the work. The ticket is the instruction set.
- **Progressive scoping**: Don't over-decompose upfront. Parent issues get full specs during Pre-Dev. Sub-issues emerge during In-Dev as the engineer discovers implementation details. See the `dev-cycle` skill for the full methodology.

### Blocking issue creation during work

When working on an issue (not just creating one), if you get stuck:

- **Hit a bug** — Create a related issue with the **Bug** label, link it via `blockedBy` on the current issue. Add a comment on the current issue noting the blocker.
- **Need human input** — Create a related issue with the **User** label, link it via `blockedBy`. Comment on the current issue noting what input is needed.

The `blockedBy` relation in Linear visually shows the dependency graph. The current issue stays In Progress — the relation itself communicates the block.

## Statuses

Linear statuses for the kylesnav team:

| Type | Status | Shorthand |
|---|---|---|
| backlog | Backlog | `backlog` |
| unstarted | Todo | `todo` |
| started | In Progress | `progress` / `wip` / `start` |
| started | In Review | `review` |
| completed | Done | `done` / `close` / `complete` |
| canceled | Canceled | `canceled` / `cancel` |
| canceled | Duplicate | `duplicate` / `dupe` |

Typical flow: Backlog → Todo → In Progress → In Review → Done

When setting status via `save_issue`, use the `state` field with the status name (e.g., `state: "In Review"`).

## Modes

### Read

Agent briefing format. Dense and structured, no prose.

1. Call `get_issue` with the identifier (request relations)
2. Call `list_comments` for the issue
3. Call `list_issues` with `parentId` to get sub-issues
4. Output:

```
## KS-42: Title
Status: In Progress | Priority: High | Labels: Feature, Scope
Project: polymarket-sim | Milestone: API Foundation | Assignee: kylesnav
Created: 2026-02-28 | Updated: 2026-03-01

### Description
[description text]

### Checklist [2/5]
1. [x] Step one
2. [x] Step two
3. [ ] Step three
4. [ ] Step four
5. [ ] Step five

### Sub-issues
- KS-43: Fix token skip (Done)
- KS-44: Add visual assertions (In Progress)

### Comments (3)
[most recent first, with author and date]

### Relations
- Blocks: KS-50
- Blocked by: KS-39
```

Omit empty sections. Checklist section only appears if description contains `- [ ]` or `- [x]` items.

### Create

Single-issue fast path. No confirmation needed.

1. **Parse `$ARGUMENTS`** — Extract title and any description content
2. **Resolve team** — Use `kylesnav` unless specified
3. **Create the issue** — Call `save_issue` with:
   - `title`: Short, imperative
   - `description`: Use Standard tier (ticket anatomy) unless the issue is clearly trivial. See Description Style below.
   - `team`: `kylesnav`
   - `assignee`: `me`
   - `priority`: `3` unless user indicates urgency
   - `labels`: From inference
   - `project`: From inference (if any)
   - `milestone`: From inference (if any)
4. **Report back** — Show the issue identifier and URL. One line, no fanfare.

### Batch Create

Triggered when the user describes multiple issues (numbered list, comma-separated, or paragraph with multiple distinct tasks).

1. **Parse all issues** from the input
2. **Present a summary table** before creating:

```
I'll create the following issues:

| # | Title | Labels | Project | Milestone |
|---|---|---|---|---|
| 1 | Add health check endpoint | Feature | polymarket-sim | API Foundation |
| 2 | Fix temperature unit conversion | Bug | polymarket-sim | — |
| 3 | Refactor data fetching layer | Feature | — | — |

Dependencies:
- #3 blocked by #2

Create all? (y/n)
```

3. **Wait for approval** — Use AskUserQuestion
4. **Create in dependency order** — Blockers first so you have their IDs for `blockedBy` fields. Each issue gets a Standard tier description by default. For large batches (5+), offer the option to create with Quick descriptions and flesh them out individually later — Standard tier in batch is slower.
5. **Report all identifiers** — List all created issues with identifiers and URLs

### Update

Read-before-write. Always fetch the issue first to confirm it exists and show the transition.

1. **Resolve identifier** — Call `get_issue`
2. **Parse changes** — Status shorthand, priority, labels, assignee, milestone, etc.
3. **Call `save_issue`** with changed fields only
4. **Report transition** — e.g., `KS-42: In Progress → Done`

### Comment

1. **Resolve identifier** — Call `get_issue` to confirm it exists
2. **Call `save_comment`** with the comment body
3. If the keyword `handoff` appears in the arguments, prefix the comment body with `**Handoff Note**\n\n` for visual distinction in Linear
4. **Report** — `Comment added to KS-43.`

### Checklist

Read-modify-write on the issue description field.

- `checklist KS-42` → Display items with 1-indexed numbers
- `check KS-42 2` → Mark item 2 done (`- [ ]` → `- [x]`)
- `uncheck KS-42 2` → Mark item 2 not done (`- [x]` → `- [ ]`)
- `check KS-42 all` → Mark all items done

Steps:
1. Call `get_issue` to get current description
2. Parse `- [ ]` and `- [x]` items from description
3. For display: show numbered list with status
4. For mutation: modify the target item(s), call `save_issue` with updated description
5. **Report** — `KS-42: Checked item 2 — "Write visual assertions"`

### Sub-issue

Sub-issues represent implementation work for executing the parent ticket. They emerge during development as the engineer discovers the concrete steps needed — they should not be pre-planned during scaffolding.

1. **Resolve parent** — Call `get_issue` on the parent identifier
2. **Create child** — Call `save_issue` with:
   - `parentId`: Parent issue's ID
   - `team`: Inherited from parent
   - `project`: Inherited from parent
   - `milestone`: Inherited from parent
   - `title`: From arguments
   - `description`: Quick tier (brief, one to three sentences — sub-issues are implementation details, not standalone specs)
   - `assignee`: `me`
3. **Report** — `Created KS-55: Fix token skip (child of KS-42)`

Sub-issues don't get the full completion checklist — they're too granular for that overhead.

### Compound Handoff

Convenience syntax: `handoff KS-42 -> KS-43 "message"`

1. **Update KS-42** — Set state to Done via `save_issue`
2. **Comment on KS-43** — Call `save_comment` with body prefixed: `**Handoff from KS-42**\n\n[message]`
3. **Report** — `KS-42 → Done. Handoff note added to KS-43.`

## Description Style

Three tiers based on issue substance. Standard is the default — use it unless you have a clear reason to go lighter or heavier.

### Quick (for trivial items)

One to three sentences. Used for bugs with obvious fixes, typos, minor config changes, and sub-issues.

> Fix typo in auth error message — "authenication" → "authentication"

### Standard (default for substantive issues)

The ticket anatomy format. Used for any issue that represents real work — features, meaningful bugs, scoping tasks, tests. This is what makes tickets agent-executable specs that Claude Code can pick up in plan mode.

```markdown
## Context
Why this work exists. What problem it solves. What's insufficient about
the current state.

## Requirements
What must be true when this is done. Concrete, testable statements.

## Acceptance Criteria
- [ ] Specific, pass/fail criterion
- [ ] Specific, pass/fail criterion
- [ ] Specific, pass/fail criterion

## Technical Approach
File paths, patterns to follow, key decisions already made.
Skip this section if the approach is obvious from the requirements.

## Completion Checklist
- [ ] Implementation complete
- [ ] Tests passing
- [ ] /simplify — review changed code for quality and efficiency
- [ ] /review — code review passes
- [ ] PR created (via finishing-a-development-branch)
```

Omit sections that don't apply. An empty section is worse than no section. The Technical Approach section is optional — skip it when the requirements make the approach obvious.

The Completion Checklist is always included for Standard-tier issues. It can be extended with issue-specific items (e.g., "Database migration tested locally", "Design approved by Kyle") — add those above the standard items.

### Detailed (on explicit request)

Used when the user says "detailed", "PRD-style", "full spec". Same as Standard but with additional sections as needed: Decision Points, Key Constraints, Migration Notes, etc.

## Completion Checklist Template

Every Standard-tier issue (and above) includes this checklist at the bottom of its description:

```markdown
## Completion Checklist
- [ ] Implementation complete
- [ ] Tests passing
- [ ] /simplify — review changed code for quality and efficiency
- [ ] /review — code review passes
- [ ] PR created (via finishing-a-development-branch)
```

The `/simplify` and `/review` items are not ceremonial — checking them off means actually running those skills and confirming the work passed. When processing checklist check-offs for these items, the expectation is that the corresponding skill was invoked during the work.

Issue-specific items go above the standard items. For example:

```markdown
## Completion Checklist
- [ ] Database migration tested locally
- [ ] API docs updated
- [ ] Implementation complete
- [ ] Tests passing
- [ ] /simplify — review changed code for quality and efficiency
- [ ] /review — code review passes
- [ ] PR created (via finishing-a-development-branch)
```

## Codebase Context

**Opt-in, not automatic.** Don't explore the codebase unless:

- You're already in a repo and the issue is about that code
- The user explicitly asks you to reference code in the issue
- The issue is a **review or audit** — when the user says "review", "audit", "check", or "make sure", explore relevant repos/files to write an informed issue with specific findings

When including codebase context:
- Reference specific files and line numbers
- Mention relevant patterns or conventions
- Keep it brief — a file path and one-line note, not a code review
- For review issues: call out specific findings

## Cross-Referencing Related Issues

When creating or updating an issue that overlaps with or depends on an existing issue:

- **Deduplicate** — If the new issue absorbs a step from an existing one, update the existing issue to remove that step and reference the new one (e.g., "Handled via KS-3")
- **Reference by identifier** — Use Linear identifiers (e.g., KS-3) in descriptions, not raw IDs
- **Clarify boundaries** — State explicitly what each issue owns vs. what's handled elsewhere

## Error Handling

| Error | Response |
|---|---|
| Issue not found | "KS-999 not found. Check the identifier." |
| No checklist in description | "KS-42 has no checklist items." |
| Invalid status shorthand | Show valid statuses from `list_issue_statuses` |
| Ambiguous identifier | Show matches, ask to specify |

## Examples

### Read an issue
```
/linear-issue KS-42
```
Shows full agent briefing: status, description, checklist progress, sub-issues, comments, relations.

### Create a single issue
```
/linear-issue Add health check endpoint to polymarket-sim API
```
Creates issue titled "Add health check endpoint" in kylesnav team, labeled Feature, project polymarket-sim. Uses Standard tier description with ticket anatomy.

### Batch create
```
/linear-issue
- Add retry logic to NOAA API calls
- Fix incorrect probability clamping at boundaries
- Add CLI flag for custom date ranges
```
Shows summary table, waits for approval, creates all three with Standard tier descriptions.

### Update status
```
/linear-issue KS-42 status:review
```
`KS-42: In Progress → In Review`

### Mark as duplicate
```
/linear-issue KS-42 status:dupe
```
`KS-42: In Progress → Duplicate`

### Compound handoff
```
/linear-issue handoff KS-42 -> KS-43 "All token work complete, visual tests next"
```
`KS-42 → Done. Handoff note added to KS-43.`

### Bare invocation
```
/linear-issue
```
Asks what the user wants to do.

## Related Skills

| Skill | Relationship |
|---|---|
| `dev-cycle` | Methodology and operating model — why tickets are structured this way |
| `linear-project` | Project-level operations — board overview, milestone scaffolding |
| `using-git-worktrees` | Create isolated worktree when starting an issue (1 issue = 1 worktree) |
| `finishing-a-development-branch` | PR creation/merge when completing an issue (1 issue = 1 PR) |
| `/simplify` | Code quality review — referenced in completion checklist |
| `/review` | Code review via subagent — referenced in completion checklist |
