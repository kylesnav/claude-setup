## Functionality
- Ask the user questions to expand on what might be a minimally express thought that they're expecting deeper output for.
- When asking the user questions or presenting choices, always use the `AskUserQuestion` tool — never list options as numbered text in the conversation. Ask clarifying questions liberally — don't assume, confirm.

## Vault structure

```
Wiki/                ← topic synthesis pages — the vault's knowledge index
Claude/              ← Claude's workspace (Findings, Freeform, Audit)
Projects/            ← active work, organized by project subfolder
Research/            ← formal research — /research output and Kyle-authored, organized by month
References/          ← clippings, PDFs, external material
Working/             ← quick capture, daily notes, scratch. Claude doesn't edit here.
Archive/             ← completed projects and their research, status: archive
```

**Research vs. Findings vs. Freeform:** Research is the vault's formal knowledge base — substantive research notes that have a permanent home. When Claude produces research via an explicit `/research` invocation, it goes directly to `Research/<Month YYYY>/` — the act of calling `/research` signals formality and permanence. Findings (`Claude/Findings/`) is a staging area for research-adjacent deliverables that may later move into Research/ — analysis produced during conversation that wasn't triggered by `/research`, or exploratory work that hasn't been triaged yet. Freeform is the default destination for everything else Claude generates in conversation (summaries, brainstorms, one-off outputs, misc files). **Routing rule:** `/research` output → `Research/`. Other substantive analysis → `Findings`. Everything else → `Freeform`. The user triages Findings and Freeform periodically.

**Subagent file creation: prohibited.** Agents spawned during a conversation (research agents, follow-up agents, any subagent) must return their output as text to the parent. Only the parent conversation writes files to the vault. This prevents routing errors and file sprawl — subagents don't have the full context to route correctly.

**Default file routing:** When a conversation produces a `.md` file to save to the vault — whether an artifact, summary, brainstorm, or any other output — save it to `Claude/Freeform/<Month YYYY>/` unless the user explicitly directs it elsewhere or the output is clearly a research deliverable (→ Findings). Apply standard frontmatter (date, tags, status, aliases) per vault rules. Freeform files should follow vault conventions but are not held to the same structural rigor as Findings — they exist to be triaged later.

**Claude/ subfolders:**
- `Claude/Findings/<Month YYYY>/` — research deliverables from live conversations, organized by month
- `Claude/Freeform/<Month YYYY>/` — catchall for all other conversation-generated files (summaries, brainstorms, misc outputs), organized by month
- `Claude/Audit/` — system audits, environment inventories, one-off diagnostic notes

---

## Wiki Protocol

`Wiki/` contains topic pages that synthesize research across the entire vault. Each page covers one subject and links to source notes in Research/, Claude/Findings/, Archive/, Projects/, and References/. Topic pages use the `research` and `wiki` tags.

**For all sessions:** When answering a question that touches a topic covered by a wiki page, read the relevant wiki page first. It provides the vault's current understanding and links to the deepest source material — including archived project research that would otherwise require explicit searching.

**After producing research:** If a /research session produces findings relevant to an existing wiki topic, update that wiki page's Source Map and, if the findings change the picture, revise the Current Understanding section. Add a Changelog entry.

**Archive is not dead storage.** Archived projects contain the vault's richest research (competitive analyses, product briefs, technical deep-dives). Wiki pages link directly into Archive — follow those links. When searching for prior work on any topic, always include Archive/ in your search scope.

**Wiki page anatomy:**
- **Current Understanding** — 2-4 paragraphs synthesizing what the vault knows
- **Key Findings** — 3-7 most important insights with wiki-links to sources
- **Source Map** — table with Source, Location, Date, and What it covers columns
- **Open Questions** — concrete, researchable gaps
- **Changelog** — reverse-chronological substantive updates

---

## Frontmatter standard

Every note must have:

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
- `tags` must include at least one from the taxonomy below
- `status` is `active` for current notes, `archive` for completed/retired
- `aliases` can be empty `[]` but the field must exist

**Clipping exception:** Files in `References/Clippings/` retain Chrome Web Clipper metadata (`title`, `source`, `author`, `published`, `created`, `description`) in addition to the standard 4 properties. These extra fields are expected and should not be flagged as stray properties.

---

## Tag taxonomy

- `ai`, `tools`, `business`, `marketing`, `career`, `finance`
- `strategy`, `plan`, `architecture`, `review`, `playbook`, `research`
- `clipping`, `wiki`
- `meta`, `audit`

If a note doesn't fit cleanly, use the closest match. Don't invent new tags — add project-specific tags to this list as needed per vault.

---

## Conventions

- **Monolithic notes.** Research notes are long-form, heavily cross-referenced. Don't atomize unless asked.
- **Table of Contents.** Long-form notes (4+ `##` sections) must include a TOC after frontmatter using `[[#Section]]` wiki-links. Don't put numbers in section headers — numbering belongs in the TOC list if needed, not the headings themselves.
- **Wiki-links are organic.** Use `[[note name]]` and `[[note name#section]]` inline where contextually relevant. No forced `## Related` appendix sections.
- **Obsidian callouts.** Use `[!note]`, `[!tip]`, `[!warning]`, `[!danger]`, `[!info]`, `[!success]`, `[!question]`, `[!quote]` sparingly — 1-3 per note max. For emphasis, not decoration.
- **Footnotes for sources.** Prefer footnotes over inline URLs when a sentence references a source. Inline URLs are fine in lists or when the link IS the content.
- **Block quotes from previous outputs.** When a finding continues or contradicts something from a previous day, pull it in with a wiki-linked block quote:

  > From [[AI Briefing — 2026-02-27#News]]:
  > OpenAI's funding round was reported at $40B — today it's confirmed at $100B+.

- **Archive notes** get `status: archive` and live in `Archive/`.
- **No templates folder, no daily notes, no inbox.** Keep it lean.
- **Prompt files.** Notes that store a copyable LLM prompt must wrap the prompt body in a quadruple-backtick code block so it renders as code, not as live markdown. Structure: frontmatter → `# Title` → 1-2 sentence description → `---` → "Copy everything below:" → ```` fence containing the prompt.
- **Stray files.** No loose `.md` files in the vault root except `CLAUDE.md`. Everything has a home.

---

## Obsidian Canvas creation

Canvas files use JSON Canvas format (`.canvas`). When creating or editing canvases:

**Format:** JSON with `nodes` (type: `text`, `group`, `file`, `link`) and `edges` arrays. Text nodes use markdown in their `text` field. Groups act as labeled containers. Edges connect nodes by `fromNode`/`toNode` with optional `fromSide`/`toSide` (`top`, `bottom`, `left`, `right`) and `label`.

**Sizing for Delightful theme:** This vault uses the Delightful theme, which has significantly larger heading sizes (h2: 2.074em, h3: 1.728em) and 2px borders with neo-brutalist offset shadows. Default canvas sizing assumptions will clip text. Use these minimums:

- Cards with an `## h2` + 2–3 lines of body: **width 500, height 200**
- Cards with an `## h2` + 4–6 lines or a code block: **width 500, height 280–320**
- Cards with an `### h3` + 2–3 lines: **width 380, height 200**
- Cards with a markdown table (4+ rows): **width 800, height 260**
- Small utility cards (1–2 lines, no heading): **width 240, height 80–180**
- Group padding: at least 50px on all sides between the group edge and its child nodes

**Color mapping:** Obsidian canvas node `color` values: `1`=red, `2`=gold, `3`=green, `4`=cyan, `5`=purple, `6`=pink. Use colors semantically to group related concepts.
