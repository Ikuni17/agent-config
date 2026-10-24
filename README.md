# Codex configuration

This private repository is the source of truth for the global Codex instructions and custom agent definitions in `codex/`.

Codex reads these absolute symlinks:

- `/Users/brad/.codex/AGENTS.md` → `/Users/brad/codex-config/codex/AGENTS.md`
- `/Users/brad/.codex/agents` → `/Users/brad/codex-config/codex/agents`

Edit either the repository paths or the linked Codex paths, then review and commit the changes here. Start a fresh Codex session after changes so it loads the updated instructions and agent definitions.

## Workflow

The primary agent coordinates the fixed roles in `codex/agents/` for repository changes. It presents an independently validated plan for approval, delegates coding, validates review findings, and handles the draft PR and Libra loop. Each role uses `gpt-6.1-sol`:

| Role | Reasoning | Responsibility |
| --- | --- | --- |
| `planner` | `xhigh` | Repository exploration and two-section operational plan |
| `plan_validator` | `high` | Independent scope, integration, and duplication checks |
| `coder` | `xhigh` | Bounded implementation, reuse, and proportional validation |
| `code_reviewer` | `xhigh` | Native Codex review plus scope and semantic duplication checks |
| `finding_validator` | `high` | Evidence, severity, and accept/reject/defer decisions |

The workflow allows three plan corrections, three local correction rounds per cycle, and three Libra-driven corrections per PR. Libra waits are capped at 10 minutes per head. The final handoff places review-focus guidance in pending inline GitHub comments for the user to submit. See `codex/AGENTS.md` for the gates, counters, fallback behavior, and existing PR policies.

Agent behavior is instruction-driven. Start a fresh session to discover changes; read-only specialist instructions inherit the session's sandbox permissions. This repository does not change the primary model, concurrency, or other `config.toml` settings. Tune each role's stable instructions in its TOML definition and keep task-specific facts in assignments.

The code reviewer embeds the native reviewer prompt from Codex CLI `0.163.0-alpha.11.2`. Its TOML comments record the installed source, exact extraction markers, length, and SHA256. Preserve that parsed instruction prefix byte-for-byte when tuning additions; verify the provenance when updating the native baseline. Custom agents are documented in the [official OpenAI documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents).

## Original backup

The original file and directory are preserved at `/Users/brad/.codex/backups/config-version-control-20261024T001141486716Z`. To restore them, first confirm the two Codex paths are still symlinks, then run:

```sh
unlink /Users/brad/.codex/AGENTS.md
unlink /Users/brad/.codex/agents
mv /Users/brad/.codex/backups/config-version-control-20261024T001141486716Z/AGENTS.md /Users/brad/.codex/AGENTS.md
mv /Users/brad/.codex/backups/config-version-control-20261024T001141486716Z/agents /Users/brad/.codex/agents
```

The remaining `.codex` configuration and session data stay outside this repository.
