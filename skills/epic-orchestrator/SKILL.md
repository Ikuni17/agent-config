---
name: epic-orchestrator
description: Coordinate planning and implementation across a GitHub epic or tracking issue in Codex or Claude Code. Use when the user wants linked issues planned or built together. Schedule independent work in parallel and manage dependent pull requests as native GitHub stacks.
---

# Epic orchestrator

The primary agent runs the installed global workflow in `AGENTS.md` or `CLAUDE.md` for each issue. Use its roles, approvals, reviews, and PR policies. Keep issue and PR state in the primary conversation. Record the dependency contracts, approved stack and merge order, author handles, worktree assignments, and reviewed revisions.

## Plan the epic

1. Resolve the epic repository and load the epic and its GitHub sub-issues, or its issue checklist when no sub-issues exist. Check each issue body and merged work to identify the remaining work. Default to one PR per issue.
2. Map dependency contracts, the issue requirements that authorize them, and the proposed stack order. Start each planning wave with issues whose upstream contracts have passed independent plan validation, and pass those contracts to downstream planners. Escalate cycles, ambiguous stack order, proposed PR splits, and changes that require new scope. Record the developer’s decision on the affected issue and keep its dependents blocked until the contract is settled. Revalidate plans affected by a change to an upstream contract.
3. Present the dependency graph and merge order alongside the unchanged, independently validated plans for one developer approval. Keep planning read-only. Use the host's global approval mechanism, with explicit approval in chat for Codex Default mode. Keep execution within the approved issue plans and scope decisions.

## Schedule work

Use issue-keyed assignments and queue ready work within the host's available agent capacity. As agents finish, record the outcome and refill their slots. A capacity error defers the queued assignment. Unrelated issues and stacks can progress when an issue and its dependents are blocked.

In Codex, use the named `agent_type`, unique task names, and the global workflow's fresh reviewer settings. Wait for available capacity with `wait_agent` and resume a completed author with `followup_task`. In Claude Code, use `Agent` with the exact `subagent_type` and the global workflow's fresh reviewer settings. Use its background wait and author resume capabilities when available. Give replacement authors the original issue assignment, approved plan, current revision, and relevant review context when the original author cannot resume.

## Execute approved work

Give each initial coder a separate worktree pinned to its recorded base and a bounded assignment with the approved plan and upstream contracts. Start an approved child layer once its parent has an initial local commit. Keep initial stack commits local until coordinator handoff and review. For dependent PRs, follow [the stack workflow](references/stacks.md) for coordination, publication, and corrections. Run independent PRs through the global workflow directly.

Assign each reviewer an absolute worktree path and exact base and head SHAs. In Claude Code, enter the existing review worktree with `EnterWorktree`, verify its root and head, and launch the fresh named `code_reviewer` there with `high --max-findings all <base>..<head>` as the bundled review arguments. Wait for the result before restoring the original checkout with `ExitWorktree`, keeping the worktree. Serialize these switches and hold the affected review if the host cannot bind it to the target checkout.

## Hand off the epic

Use `Part of #<epic>` alongside the global issue reference. Use `Closes #<epic>` only for the designated final PR when merging in the approved order completes the epic. Flag issue and epic closure for verification when a PR targets a stack branch. Report the merge order, any blocked items, and each PR's global workflow handoff. Leave merges to the developer.
