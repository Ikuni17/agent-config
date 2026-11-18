---
name: epic-orchestrator
description: >-
  Orchestrate planning and implementation across the sub-issues of a GitHub epic using host-native parallel subagents while keeping the orchestrator context small. Use whenever the user points at an epic, parent issue, tracking issue, or multi-issue body of work and wants its linked tasks planned or built as a set. Map dependencies, plan in reviewed waves, present an approval gate, implement issues in parallel worktrees, then stabilize assigned PRs from a shared stack coordinator. Supports Codex and Claude Code through host-specific adapters.
---

# Epic Orchestrator

Act only as the orchestrator. Delegate exploration, planning, implementation, and review; route feedback; and assemble outputs. Do not author issue plans or code.

## Scope invariant

For each issue, permit only the smallest changeset necessary to satisfy its issue text and the explicit contracts it must provide to or consume from other issues in the epic. Every planned change and implementation diff must be traceable to one of those requirements.

If an issue does not authorize work required by an epic contract, or any agent proposes work outside that boundary, block the affected issue and ask the developer for clarification. State the exact gap and proposed disposition of removal or the smallest necessary scope addition. Do not add that work to a plan or implementation until the developer explicitly approves it and the approval is recorded verbatim. Epic-level execution approval does not approve an unraised scope addition.

Create new pull requests as drafts. If a pull request is ready for review, treat that state as an intentional developer action; no agent may convert it back to draft.

## Select the host adapter

Determine the host from explicit system instructions and the available agent tools, not from this skill's filesystem path. The same directory may be discovered through a symlink.

Read exactly one adapter completely before taking workflow actions:

- Codex: `references/codex.md`
- Claude Code: `references/claude.md`

If the host is unclear or unsupported, stop and state which orchestration capabilities could not be identified. Do not guess tool names or lifecycle behavior.

After the adapter permits the run, read `references/workflow.md` completely. Apply the installed `Global workflow` in `AGENTS.md` or `CLAUDE.md` and use its `planner`, `plan_validator`, `coder`, `code_reviewer`, and `finding_validator` roles.

The host adapter controls agent spawning, handles, concurrency, follow-ups, permissions, result visibility, and worktree isolation. The shared workflow controls dependency ordering, artifacts, review independence, approval, PR shape, and safety. Follow the installed Global workflow for role assignments, review, and PR policies. When the host adapter and shared workflow conflict, follow the adapter for host mechanics without weakening the shared safety and approval gates.

Keep detailed work in scratch files and only compact status in the orchestrator's context. Use a fresh independent agent for every review. Route revisions to the original author when the host supports it, or use the adapter's artifact-seeded replacement-author fallback.

This is a single-session workflow. If the session ends, active orchestration stops.
