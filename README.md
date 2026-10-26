# Codex configuration

This private repository is the source of truth for the global Codex instructions and custom agent definitions in `codex/`.

Codex reads these absolute symlinks:

- `/Users/brad/.codex/AGENTS.md` → `/Users/brad/codex-config/codex/AGENTS.md`
- `/Users/brad/.codex/agents` → `/Users/brad/codex-config/codex/agents`

Edit either the repository paths or the linked Codex paths, then review and commit the changes here. Start a fresh Codex session after changes so it loads the updated instructions and agent definitions.

## Workflow

The primary agent coordinates the fixed roles in `codex/agents/` for repository changes. When building or revising an issue plan, or working in Plan mode, it follows the planning guidelines and presents an independently validated plan for approval before coding. Otherwise, it acts on proposed changes and comments through coding and proportional local review without a formal plan or extra approval. Questions are answered directly. Each role uses `gpt-6.1-sol`:

| Role | Reasoning | Responsibility |
| --- | --- | --- |
| `planner` | `xhigh` | Repository exploration and two-section operational plan |
| `plan_validator` | `high` | Independent scope, integration, and duplication checks |
| `coder` | `xhigh` | Bounded implementation, reuse, and proportional validation |
| `code_reviewer` | `xhigh` | Native Codex review plus scope and semantic duplication checks |
| `finding_validator` | `high` | Evidence, severity, and accept/reject/defer decisions |

The workflow allows three plan corrections, three local correction rounds per cycle, and three Libra-driven corrections per PR. Libra is optional: repositories without it can become ready for human review after local review completes, with an explicit local-review outcome. Repositories that use Libra retain the 10-minute wait per head and draft fallback for failed, unavailable, or incomplete reviews; unresolved applicability also preserves the draft. Both completion paths place review-focus guidance in pending inline GitHub comments for the user to submit. See `codex/AGENTS.md` for applicability checks, gates, counters, fallback behavior, and existing PR policies.

Agent behavior is instruction-driven. Start a fresh session to discover changes; read-only specialist instructions inherit the session's sandbox permissions. This repository does not change the primary model, concurrency, or other `config.toml` settings. Tune each role's stable instructions in its TOML definition and keep task-specific facts in assignments.

Each role uses short bullet points grouped by responsibility. The code reviewer reads the current [native Codex review rubric](https://github.com/openai/codex/blob/main/codex-rs/prompts/templates/review/rubric.md) before each review and applies the global scope and reuse checks alongside it. Source access is required; the reviewer reports a blocker if it cannot read the rubric. Custom agents are documented in the [official OpenAI documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents).

## Original backup

The original file and directory are preserved at `/Users/brad/.codex/backups/config-version-control-20261024T001141486716Z`. To restore them, first confirm the two Codex paths are still symlinks, then run:

```sh
unlink /Users/brad/.codex/AGENTS.md
unlink /Users/brad/.codex/agents
mv /Users/brad/.codex/backups/config-version-control-20261024T001141486716Z/AGENTS.md /Users/brad/.codex/AGENTS.md
mv /Users/brad/.codex/backups/config-version-control-20261024T001141486716Z/agents /Users/brad/.codex/agents
```

The remaining `.codex` configuration and session data stay outside this repository.
