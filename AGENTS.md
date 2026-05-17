# claude-setup

Machine setup orchestrator for Kyle's Delightful development environment.

## Source of Truth

This repo coordinates setup instructions and vault templates. Platform theme/config files are sourced from their standalone repos under `kylesnav/*`.

## Validate

- Review `README.md`, `SETUP.md`, and `CLAUDE.md` together when changing setup flow.
- Verify every raw GitHub URL points to an existing file.
- Preserve the safety model: detect, diff, propose, and ask before mutating user machine files.

## Editing Rules

- Never instruct agents to overwrite existing dotfiles or vault files blindly.
- Shell config instructions should append/source snippets, not replace whole files.
- Keep Delightful repo references aligned with the standalone repos.
- Treat `obsidian-vault-template/` as user-facing scaffold; changes there should be deliberate.

## Release Notes

There is no build step. Changes are docs/templates/skills/agents. PR summaries should call out any setup commands or destination paths that changed.
