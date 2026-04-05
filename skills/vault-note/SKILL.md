---
name: vault-note
description: Create Obsidian vault notes with correct frontmatter, taxonomy, and formatting conventions
allowed-tools: "Bash Read Write Edit Glob Grep"
---

# Vault Note

Create notes for Kyle's Obsidian vault following all established conventions.

## Frontmatter

Every note must include:

```yaml
---
date: YYYY-MM-DD
tags:
  - <tag>
status: active | archive
aliases: []
---
```

- `date` is the creation or last-major-edit date
- `tags` must include at least one from the taxonomy
- `status` is `active` for current notes, `archive` for completed/retired
- `aliases` can be empty `[]` but the field must exist

## Tag Taxonomy

Use only these tags: `ai`, `tools`, `business`, `marketing`, `career`, `finance`, `strategy`, `plan`, `architecture`, `review`, `playbook`, `research`, `delightful`, `github`, `braze`, `rosetta`, `clipping`, `briefing`, `meta`, `audit`

If a note doesn't fit cleanly, use the closest match. Don't invent new tags.

## Placement Rules

| Content type | Location |
|---|---|
| Formal research — `/research` output or Kyle-authored | `Research/<Month YYYY>/` |
| Research-adjacent deliverables staging (not from `/research`) | `Claude/Findings/<Month YYYY>/` |
| Active project work | `Projects/<project-name>/` |
| Clippings, PDFs, external material | `References/` |
| Topic synthesis pages (wiki) | `Wiki/` |
| Completed/retired notes | `Archive/` (set `status: archive`) |

## Table of Contents

Notes with 4 or more `##` sections must include a numbered TOC after the frontmatter using `[[#Section]]` wiki-links. Do NOT put numbers in section headers — numbering belongs in the TOC list only.

## Pre-Write Vault Check (CLI)

Before writing a note, use the Obsidian CLI to improve placement and linking (requires Obsidian running — skip if unavailable):

1. **Find linking candidates** — `obsidian search query="<topic keywords>" limit=10` to discover notes worth wiki-linking
2. **Check for orphans** — `obsidian backlinks file="<similar note>"` to see what links into related content and mirror those patterns
3. **Verify placement** — `obsidian files folder="<target folder>"` to confirm the destination folder exists and see sibling notes

Fall back to Glob/Grep if the CLI is unavailable.

4. **Wiki relevance** — If the note is tagged `research`, check `Wiki/` for a topic page that should link to this note. Flag it for the user if a wiki update seems warranted. Do not auto-edit wiki pages from vault-note — that's the research skill's job or a manual session.

## Formatting Conventions

- **Monolithic notes.** Notes are long-form and heavily cross-referenced. Don't atomize.
- **Wiki-links are organic.** Use `[[note name]]` and `[[note name#section]]` inline where contextually relevant. No forced `## Related` appendix sections.
- **Obsidian callouts.** Use `[!note]`, `[!tip]`, `[!warning]`, `[!danger]`, `[!info]`, `[!success]`, `[!question]`, `[!quote]` sparingly — 1–3 per note max.
- **Footnotes for sources.** Prefer footnotes over inline URLs when a sentence references a source.[^1] Inline URLs are fine in lists or when the link IS the content.
- **No inline `#tags` in runner/automated output.** Inline tags are welcome in conversational notes.
- **Block quotes from previous outputs.** When continuing or contradicting a previous note, pull it in with a wiki-linked block quote:
  > From [[AI Briefing — 2026-02-27#News]]:
  > Content from that note...
- **No loose files in vault root.** Everything has a home.
