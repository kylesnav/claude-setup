# Claude Setup

A complete, opinionated environment for working with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — custom skills, agents, an Obsidian vault template, and a setup orchestration system that detects, proposes, and applies changes across machines.

This is a personal setup. Fork it and adapt to your own tools, paths, and workflows.

## What's in here

**Skills** — slash commands that extend Claude Code with domain-specific workflows:

| Skill | What it does |
|-------|-------------|
| `/dev-cycle` | EPD development methodology — milestones, ticket anatomy, progressive scoping |
| `/linear-issue` | Full Linear issue lifecycle — create, read, update, comment, sub-issues, handoffs |
| `/linear-project` | Project board overview and milestone scaffolding |
| `/research` | Web research pipeline that outputs vault-formatted monolithic notes |
| `/vault-note` | Create Obsidian vault notes with correct frontmatter and conventions |

**Agents** — autonomous subagents for specific tasks:

| Agent | What it does |
|-------|-------------|
| `vault-agent` | Obsidian vault operations — health audits, tag hygiene, linking, content quality |

**Obsidian Vault Template** — a starter vault with pre-configured settings, folder structure, and a CLAUDE.md with conventions for how Claude interacts with the vault. Theme and plugins are pulled fresh from GitHub at setup time.

**Environment Orchestration** — a `CLAUDE.md` at the repo root that instructs Claude to detect your environment, report what's installed/missing/outdated, propose changes with diffs, and let you accept each one individually. See [SETUP.md](SETUP.md) for the full checklist.

## The Stack

This setup is built around the [Delightful Design System](https://github.com/kylesnav/delightful-design-system) — a neo-brutalist theme that runs across the entire dev environment:

| Tool | Theme repo |
|------|-----------|
| Ghostty | [delightful-ghostty](https://github.com/kylesnav/delightful-ghostty) |
| Starship | [delightful-starship](https://github.com/kylesnav/delightful-starship) |
| Shell (tmux + zsh) | [delightful-shell](https://github.com/kylesnav/delightful-shell) |
| VS Code | [delightful-vscode](https://github.com/kylesnav/delightful-vscode) |
| Obsidian | [obsidian-delightful](https://github.com/kylesnav/obsidian-delightful) |
| iTerm2 | [delightful-iterm2](https://github.com/kylesnav/delightful-iterm2) |
| Claude Code | [delightful-claude-plugin](https://github.com/kylesnav/delightful-claude-plugin) |

## Install

Copy skills and agents into `~/.claude/`:

```sh
cp -r skills/* ~/.claude/skills/
cp -r agents/* ~/.claude/agents/
```

## New Obsidian Vault

```sh
cp -r obsidian-vault-template /path/to/new-vault
```

Open in Obsidian — folder structure and settings are ready. Theme and plugins are downloaded during the setup flow.

## Full Machine Setup

Clone this repo, open Claude Code in it, and Claude walks you through the rest — detecting what's installed, showing what's missing, and applying changes one at a time. See [SETUP.md](SETUP.md) for the manual checklist.

## License

[MIT](LICENSE)
