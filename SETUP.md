# New Machine Setup

Checklist for setting up a new machine with the full Delightful environment. All configs are pulled directly from GitHub — no need to clone repos you aren't developing on.

**Preferred workflow:** Clone this repo, open Claude Code in it, and let it detect what's installed and walk you through the rest. The CLAUDE.md has full environment detection instructions.

## Prerequisites

```sh
brew install --cask font-jetbrains-mono-nerd-font  # Nerd Font for Starship + tmux
brew install starship                                # Starship prompt
touch ~/.hushlogin                                   # Suppress "Last login" message
```

## 1. Working directory structure

```sh
mkdir -p ~/Desktop/Working/{Base,Github,Playground}
```

## 2. This repo

```sh
cd ~/Desktop/Working/Github
git clone git@github.com:kylesnav/claude-setup.git
```

## 3. Terminal — Ghostty

```sh
mkdir -p ~/.config/ghostty/themes
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-ghostty/main/themes/delightful-light -o ~/.config/ghostty/themes/delightful-light
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-ghostty/main/themes/delightful-dark -o ~/.config/ghostty/themes/delightful-dark
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-ghostty/main/config -o ~/.config/ghostty/config
```

*(optional)* Shaders — vignette/bloom post-processing:
```sh
mkdir -p ~/Library/Application\ Support/com.mitchellh.ghostty/shaders
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-ghostty/main/shaders/vignette.glsl -o ~/Library/Application\ Support/com.mitchellh.ghostty/shaders/vignette.glsl
```

## 4. Terminal — Starship prompt

```sh
mkdir -p ~/.config
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-starship/main/starship.toml -o ~/.config/starship.toml
```

Add to `~/.zshrc`:
```sh
eval "$(starship init zsh)"
```

## 5. Terminal — Shell (zsh + tmux)

Zsh snippet — either curl it to a local path and source it, or clone the repo:
```sh
mkdir -p ~/.config/delightful
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-shell/main/zshrc-snippet -o ~/.config/delightful/zshrc-snippet
# Add to ~/.zshrc:
#   source ~/.config/delightful/zshrc-snippet
```

*(optional)* tmux — persistent sessions with Delightful status bar:
```sh
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-shell/main/tmux.conf -o ~/.tmux.conf
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
# Then press prefix+I inside tmux to install plugins
```

*(optional)* tmux auto-attach — each Ghostty window gets a persistent tmux session:
```sh
mkdir -p ~/.local/bin
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-shell/main/tmux-auto-attach.sh -o ~/.local/bin/tmux-auto-attach
chmod +x ~/.local/bin/tmux-auto-attach
```

## 6. Terminal — iTerm2 *(alternative to Ghostty)*

Skip this if using Ghostty. Download and open the color profiles:
```sh
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-iterm2/main/colors/Delightful-Light.itermcolors -o /tmp/Delightful-Light.itermcolors && open /tmp/Delightful-Light.itermcolors
curl -sL https://raw.githubusercontent.com/kylesnav/delightful-iterm2/main/colors/Delightful-Dark.itermcolors -o /tmp/Delightful-Dark.itermcolors && open /tmp/Delightful-Dark.itermcolors
```

Then set up automatic light/dark switching in iTerm2 Settings > Profiles > Colors.

## 7. VS Code

The theme needs to be built from source, so clone the repo:
```sh
cd ~/Desktop/Working/Github
git clone git@github.com:kylesnav/delightful-vscode.git
cd delightful-vscode
npx @vscode/vsce package
code --install-extension delightful-theme-*.vsix
```

Select "Delightful Light" or "Delightful Dark" in VS Code color theme settings.

## 8. Obsidian vault

Scaffold the vault from the template:
```sh
cp -r ~/Desktop/Working/Github/claude-setup/obsidian-vault-template ~/Desktop/Working/Base
```

Install the Delightful theme:
```sh
mkdir -p ~/Desktop/Working/Base/.obsidian/themes/Delightful
curl -sL https://raw.githubusercontent.com/kylesnav/obsidian-delightful/main/theme.css -o ~/Desktop/Working/Base/.obsidian/themes/Delightful/theme.css
curl -sL https://raw.githubusercontent.com/kylesnav/obsidian-delightful/main/manifest.json -o ~/Desktop/Working/Base/.obsidian/themes/Delightful/manifest.json
```

Install community plugins:
```sh
for plugin in mgmeyers/obsidian-style-settings FlorianWoelki/obsidian-iconize isitwho/easylink mgmeyers/obsidian-copy-block-link; do
  id=$(gh release download --repo "$plugin" --pattern 'manifest.json' --output - 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")
  mkdir -p ~/Desktop/Working/Base/.obsidian/plugins/"$id"
  gh release download --repo "$plugin" --dir ~/Desktop/Working/Base/.obsidian/plugins/"$id" --pattern 'main.js' --pattern 'manifest.json' --pattern 'styles.css' --clobber 2>/dev/null
done
```

Copy plugin config data from template:
```sh
cp ~/Desktop/Working/Github/claude-setup/obsidian-vault-template/.obsidian/plugins/obsidian-icon-folder/data.json ~/Desktop/Working/Base/.obsidian/plugins/obsidian-icon-folder/data.json
cp ~/Desktop/Working/Github/claude-setup/obsidian-vault-template/.obsidian/plugins/obsidian-style-settings/data.json ~/Desktop/Working/Base/.obsidian/plugins/obsidian-style-settings/data.json
```

Open `~/Desktop/Working/Base` in Obsidian.

## 9. Claude Code

```sh
# Skills and agents from this repo
cp -r ~/Desktop/Working/Github/claude-setup/skills/* ~/.claude/skills/
cp -r ~/Desktop/Working/Github/claude-setup/agents/* ~/.claude/agents/

# Delightful plugin
claude plugin install kylesnav/delightful-claude-plugin

# Set Claude Code theme to match terminal
# Run /config in Claude Code → set theme to light-ansi or dark-ansi
```

## Post-setup

- [ ] Verify Ghostty theme loads (restart Ghostty)
- [ ] Verify Starship prompt renders (new shell tab)
- [ ] Verify tmux status bar *(if installed)* (start tmux session)
- [ ] Verify VS Code theme (open VS Code)
- [ ] Verify Obsidian vault opens with Delightful theme
- [ ] Verify Claude Code skills load (`/help` to check available skills)
