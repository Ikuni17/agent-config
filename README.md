# Codex configuration

This private repository is the source of truth for the global Codex instructions and custom agent definitions in `codex/`.

Codex reads these absolute symlinks:

- `/Users/brad/.codex/AGENTS.md` → `/Users/brad/codex-config/codex/AGENTS.md`
- `/Users/brad/.codex/agents` → `/Users/brad/codex-config/codex/agents`

Edit either the repository paths or the linked Codex paths, then review and commit the changes here. Start a fresh Codex session after changes so it loads the updated instructions and agent definitions.

The original file and directory are preserved at `/Users/brad/.codex/backups/config-version-control-20261024T001141486716Z`. To restore them, first confirm the two Codex paths are still symlinks, then run:

```sh
unlink /Users/brad/.codex/AGENTS.md
unlink /Users/brad/.codex/agents
mv /Users/brad/.codex/backups/config-version-control-20261024T001141486716Z/AGENTS.md /Users/brad/.codex/AGENTS.md
mv /Users/brad/.codex/backups/config-version-control-20261024T001141486716Z/agents /Users/brad/.codex/agents
```

The remaining `.codex` configuration and session data stay outside this repository.
