---
name: research
description: "Web research pipeline outputting vault-formatted monolithic notes. Use this skill whenever the user asks for research, investigation, strategic analysis, competitive analysis, deep dives, or any substantive question requiring multi-source synthesis — even if they don't say 'research' explicitly. Trigger on phrases like 'look into', 'what's the landscape for', 'dig into', 'find out about', 'I need to understand', 'what are my options for', or any ask that clearly requires going beyond what's in memory or the vault. If the answer requires more than one search to do well, this skill applies."
allowed-tools: "Bash WebSearch WebFetch Read Write Edit Glob Grep Agent"
---

# Research

Conduct deep, multi-source research and produce a vault-formatted monolithic note. This skill is the complete playbook — methodology, workflow, and output formatting in one place.

## Research methodology

Research means iterative deepening across multiple angles, not a single pass of broad queries.

### Phase 1: Scope and scan (before any web research)

1. **Clarify scope** — If the topic is ambiguous, use AskUserQuestion to narrow it. Don't assume.
2. **Check the wiki** — List files in `Wiki/` and read any topic pages relevant to this research. Wiki pages provide:
   - **Current Understanding** — the vault's existing synthesis on this subject
   - **Source Map** — links to the deepest prior work across all vault locations including Archive/
   - **Open Questions** — concrete gaps that can inform your research angles
   If a relevant wiki page exists, use it as your baseline. Don't duplicate research that's already been done — extend it.
3. **Scan the vault** — Find existing notes for context and wiki-linking. Search broadly — include Archive/ and Projects/ subfolders, not just Research/ and Claude/Findings/. Prefer the Obsidian CLI when Obsidian is running:
   - `obsidian search query="<topic>" limit=10`
   - `obsidian backlinks file="<relevant note>"`
   - `obsidian tags name="<tag>" verbose`
   - Fall back to Glob/Grep if the CLI is unavailable
4. **Draft a research plan** — Identify 4-6 distinct angles to cover, informed by wiki context and open questions. These become your parallel agent assignments.

### Phase 2: Parallel deep search

Launch **5+ parallel research agents**, each covering a distinct angle. Every agent should run multiple searches (not one query per agent). Agents should:

- Search across source types: official docs, news articles, blog posts, forums, academic papers, GitHub repos, industry reports
- Follow up on first-pass findings — if a search surfaces a key player, paper, or event, dig deeper on that specifically
- Note conflicting information explicitly rather than silently picking one version
- Record source URLs for footnotes
- **Return findings as text — do NOT write files to the vault.** Only the parent conversation writes the final note. Agents that write their own files create sprawl and bypass routing rules.

Aim for **breadth first, then depth** on the most promising threads.

### Phase 3: Gap analysis and follow-up

After first-pass agents return:

- Identify gaps: angles that came back thin, claims with only one source, questions the first pass raised but didn't answer
- Spawn **follow-up agents** targeting those gaps specifically (same rule: agents return text, never write files)
- Do not synthesize prematurely — if the evidence base is thin, keep researching

The research phase should be substantive. For major asks, expect multiple rounds of agent work before synthesis is appropriate.

### Phase 4: Synthesis

Only after the evidence base across all agents is sufficient:

- Cross-reference claims across sources
- Identify consensus vs. disagreement
- Flag where evidence is strong vs. speculative
- Synthesize into a single monolithic note (not a list of agent summaries)

## Output

Write the note to `Research/<Month YYYY>/<Topic Name>.md` in the vault. The `/research` skill always routes to `Research/` — it signals formal, permanent research. `Claude/Findings/` is a staging area for research-adjacent deliverables that weren't triggered by `/research`.

### Formatting

Follow all vault-note conventions from the **vault-note** skill (`.claude/skills/vault-note/SKILL.md`): frontmatter template, tag taxonomy, TOC rules, callout limits, footnotes, monolithic format, and wiki-linking style. The essentials:

**Frontmatter** (required on every note):
```yaml
---
date: YYYY-MM-DD
tags:
  - research
  - <additional relevant tags>
status: active
aliases: []
---
```

**Tag taxonomy:** `ai`, `tools`, `business`, `marketing`, `career`, `finance`, `strategy`, `plan`, `architecture`, `review`, `playbook`, `research`, `delightful`, `github`, `braze`, `rosetta`, `clipping`, `briefing`, `meta`, `audit`

**Table of Contents:** Required if 4+ `##` sections. Use `[[#Section]]` wiki-links. No numbers in section headers — numbering in the TOC list only.

**Wiki-links:** Use `[[note name]]` and `[[note name#section]]` inline where contextually relevant. Link to vault notes discovered during the scan phase. No forced `## Related` appendix sections.

**Callouts:** Use `[!note]`, `[!tip]`, `[!warning]`, `[!info]`, `[!question]` sparingly — 1-3 per note max. For emphasis, not decoration.

**Footnotes for sources:** Prefer footnotes over inline URLs when a sentence references a source. Inline URLs are fine in lists or when the link IS the content.

**Monolithic format:** Research notes are long-form and heavily cross-referenced. Don't atomize unless asked.

**No inline `#tags`** in the note body — tags go in frontmatter only.

### Quality bar

- Minimum 5 distinct sources (more for substantive topics)
- Cross-reference claims across sources where possible
- Note conflicting information explicitly — don't silently resolve disagreements
- Include dates/versions for time-sensitive technical content
- Flag gaps honestly — if you couldn't access something or a source was unavailable, say so
- Every major claim should have a footnoted source
- If the evidence base is thin after research, say so rather than padding with filler

### What good research output looks like

The note should read like an informed briefing from someone who spent real time on the topic — not like a reformatted search results page. The structure should follow the topic's natural logic, not the order you happened to find things. Lead with the most important insights, not the most obvious ones.

### Phase 5: Wiki integration

After writing the Research note:

1. List files in `Wiki/` and check if a relevant topic page exists
2. If yes: add the new Research note to the topic page's Source Map table with date and a one-line summary of what it covers. If the findings materially change the picture, revise the Current Understanding section. Add a Changelog entry.
3. If no relevant topic exists but the research opens a new subject area, note this for the user — don't auto-create topic pages (Kyle curates the topic list)

## Failure modes to avoid

- **Single-pass searching:** One round of broad queries is not research. First-pass findings should generate follow-up queries.
- **Premature synthesis:** Polished formatting on shallow content is worse than rough notes on deep content.
- **Silent abandonment:** If a source fails or a search returns nothing, retry with alternative terms or approaches. Do not silently move on.
- **Agent-summary format:** The output is a synthesized note, not a compilation of "Agent 1 found X, Agent 2 found Y."
- **Shallow breadth:** Five agents each running one search is not better than one agent running five searches. Depth matters.