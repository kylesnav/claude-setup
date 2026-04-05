# Claude Setup

This repo is the single source of truth for the user's dev environment across machines.

## On session start

When a conversation begins in this repo, run the full setup workflow below. Do NOT execute any changes until the user explicitly approves each one.

### Step 1 — Detect

Check every component listed in "Environment components" below. For each one:
- Read the live installed file (if it exists)
- Fetch the source from the GitHub repo (via `curl` or GitHub MCP)
- Compare them

### Step 2 — Report

Present a status dashboard showing every component and its status:
- **Installed** — file exists and matches the source repo
- **Outdated** — file exists but differs from the source repo (show what changed)
- **Missing** — component is not installed
- **N/A** — not applicable to this machine (e.g., iTerm2 on a Ghostty-only setup)

### Step 3 — Propose

For each component that is missing or outdated, present the exact proposed change:
- **File path** where the change will be made
- **What will be written** — full content for new files, diff for existing files
- **For shell configs (`~/.zshrc`, etc.):** Show what will be appended, not the whole file. NEVER propose replacing the entire file.
- **For existing configs:** Show the diff between current and proposed. Highlight any local customizations that would be preserved or affected.

Present all proposals together as a single overview so the user can see the full scope before approving anything.

### Step 4 — Accept

Use `AskUserQuestion` to let the user approve or reject each proposed change individually. Only execute approved changes. If the user rejects a change, skip it and move on.

## Safety rules — NEVER overwrite blindly

This setup runs on machines that may already have work-specific config (corporate VPN settings, `use bedrock` directives in Claude config, custom shell aliases, environment variables, etc.). Every write operation must be context-aware:

- **Files that don't exist yet:** Safe to create, but still show the user what will be written before doing it.
- **Files that already exist:** Read them first. Diff against what you'd write. Show the user what would change and ask before overwriting. If the existing file has local customizations that aren't in the template, **preserve them**.
- **Shell config (`~/.zshrc`, `~/.zprofile`, etc.):** NEVER replace wholesale. These accumulate machine-specific config over time (corporate tooling, `PATH` additions, `eval` lines, work-specific env vars). Only **append** the Delightful source line if it isn't already there. Flag any conflicts you notice.
- **Ghostty/Starship/tmux configs:** These are more self-contained, but still check for local modifications before replacing. If the user has added custom keybinds or work-specific settings, surface them.
- **Obsidian vault:** If `~/Desktop/Working/Base` already exists, do NOT overwrite it — it may contain notes. Only scaffold into an empty or nonexistent directory. For updates to theme/plugins in an existing vault, target the specific `.obsidian/` subdirectories.
- **Claude Code skills/agents:** Compare file contents before copying. If a skill has been locally modified, flag it rather than silently reverting.
- **Plugin config data (`data.json`):** Only copy from template if the destination file doesn't already exist. Existing config may have been customized in the live vault.

**General principle:** Treat every destination as potentially containing work the user cares about. Detect first, show what would change, get explicit approval, then act.

## Environment components

### Ghostty (terminal emulator)

| What | Source | Destination |
|------|--------|-------------|
| Theme files | `kylesnav/delightful-ghostty` `themes/delightful-light`, `themes/delightful-dark` | `~/.config/ghostty/themes/` |
| Config | `kylesnav/delightful-ghostty` `config` | `~/.config/ghostty/config` |
| Shaders *(optional)* | `kylesnav/delightful-ghostty` `shaders/*.glsl` | `~/Library/Application Support/com.mitchellh.ghostty/shaders/` |

**Check:** `~/.config/ghostty/config` exists and `~/.config/ghostty/themes/delightful-light` exists.

### Starship (prompt)

| What | Source | Destination |
|------|--------|-------------|
| Config | `kylesnav/delightful-starship` `starship.toml` | `~/.config/starship.toml` |

**Check:** `~/.config/starship.toml` exists. Verify `starship` is installed (`which starship`).

### Shell (zsh + tmux)

| What | Source | Destination |
|------|--------|-------------|
| Zsh snippet | `kylesnav/delightful-shell` `zshrc-snippet` | Sourced from `~/.zshrc` |
| tmux config *(optional)* | `kylesnav/delightful-shell` `tmux.conf` | `~/.tmux.conf` |
| tmux auto-attach *(optional)* | `kylesnav/delightful-shell` `tmux-auto-attach.sh` | `~/.local/bin/tmux-auto-attach` |

**Check:** `~/.zshrc` exists and contains a `source` line referencing `delightful-shell` or `zshrc-snippet`. For tmux, check if `~/.tmux.conf` exists.

### iTerm2 *(alternative to Ghostty)*

| What | Source | Destination |
|------|--------|-------------|
| Color profiles | `kylesnav/delightful-iterm2` `colors/*.itermcolors` | Imported via `open` command |

**Check:** Only relevant if iTerm2 is installed. Check `ls /Applications/iTerm.app` or `ls "$HOME/Applications/iTerm.app"`.

### VS Code

| What | Source | Destination |
|------|--------|-------------|
| Theme extension | `kylesnav/delightful-vscode` | Built with `vsce package`, installed with `code --install-extension` |

**Check:** `code --list-extensions 2>/dev/null | grep -i delightful` to see if the theme is installed.

### Obsidian vault

| What | Source | Destination |
|------|--------|-------------|
| Vault scaffold | `obsidian-vault-template/` (this repo) | `~/Desktop/Working/Base/` |
| Theme | `kylesnav/obsidian-delightful` `theme.css` + `manifest.json` | `<vault>/.obsidian/themes/Delightful/` |
| Plugins | Community plugin repos (see below) | `<vault>/.obsidian/plugins/<id>/` |

**Community plugins to install:**

| Plugin ID | Repo |
|-----------|------|
| `obsidian-style-settings` | `mgmeyers/obsidian-style-settings` |
| `obsidian-icon-folder` | `FlorianWoelki/obsidian-iconize` |
| `easylink` | `isitwho/easylink` |
| `obsidian-copy-block-link` | `mgmeyers/obsidian-copy-block-link` |

Plugin config data (icon mappings, style settings) is stored in `obsidian-vault-template/.obsidian/plugins/*/data.json` — copy these after downloading plugin code.

**Check:** `~/Desktop/Working/Base/.obsidian/appearance.json` exists. Check if theme dir exists at `~/Desktop/Working/Base/.obsidian/themes/Delightful/theme.css`. Check if plugins exist.

### Claude Code

| What | Source | Destination |
|------|--------|-------------|
| Skills | `skills/` (this repo) | `~/.claude/skills/` |
| Agents | `agents/` (this repo) | `~/.claude/agents/` |
| Plugin | `kylesnav/delightful-claude-plugin` | Installed via `claude plugin install` |

**Check:** Compare files in `~/.claude/skills/` and `~/.claude/agents/` against this repo. Check `claude plugin list 2>/dev/null` for delightful-claude-plugin.

### Working directory

```
~/Desktop/Working/
├── Base/       ← Obsidian vault
├── Github/     ← Git repos
└── Playground/ ← Scratch/experiments
```

**Check:** All three directories exist.

## How to install components

For components sourced from GitHub repos, use `curl` to pull raw files directly — no need to clone repos that aren't being actively developed:

```sh
# Example: pull a file from a GitHub repo
curl -sL https://raw.githubusercontent.com/OWNER/REPO/main/PATH -o DESTINATION

# Example: download latest release asset from a GitHub repo (for Obsidian plugins)
gh release download --repo OWNER/REPO --pattern 'main.js' --dir DESTINATION
gh release download --repo OWNER/REPO --pattern 'manifest.json' --dir DESTINATION
gh release download --repo OWNER/REPO --pattern 'styles.css' --dir DESTINATION 2>/dev/null  # not all plugins have styles
```

For items in this repo (skills, agents, vault template), copy directly from the local checkout.

## Prerequisites

Before installing components, ensure these are available:

```sh
brew install --cask font-jetbrains-mono-nerd-font  # Nerd Font for Starship + tmux
brew install starship                                # Starship prompt
touch ~/.hushlogin                                   # Suppress "Last login" message
```
