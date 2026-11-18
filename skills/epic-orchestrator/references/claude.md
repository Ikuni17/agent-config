# Claude Code adapter

Use this adapter only when system instructions identify Claude Code or the available delegation surface is Claude Code's agent/subagent tool.

## Mode and permissions

If the active Claude permission or planning mode forbids writes outside the repository, stop because the workflow requires scratch artifacts. Keep repository files observational through Phase 4.

The orchestrator writes scratch artifacts from the installed roles' returned results. Use the installed role definitions' model and effort settings.

## Agent operations

- Spawn one installed role per scheduled unit using `Agent` with the role name as `subagent_type` and an issue-keyed description or label. Use the Global workflow's tool mechanics for fresh instances and waits.
- Record the returned agent ID when the tool exposes one and resume that author for revisions.
- Use the tool's concurrent-agent limit as queue capacity and enforce workflow barriers before advancing.
- Store required role outputs in the scratch artifacts and keep compact status in the parent context.
- Create every reviewer as a new subagent that did not author the artifact.

If the current Claude surface cannot resume a particular completed author, use the shared replacement-author fallback. Give the replacement only the issue artifacts, upstream contracts, repository rules, and review findings it needs.

## Worktrees

For initial implementation, use Claude's native `isolation: "worktree"` agent option only when the available tool can bind the exact branch and recorded parent commit, return the worktree's absolute path, and allow the orchestrator to detach it at the coordinator barrier. Otherwise create an explicit Git worktree from the recorded parent commit. Before handoff, reuse a registered worktree or attach an existing branch without `-b`; never recreate or reset it. Verify the checked-out branch and ancestry, then pass the absolute path, branch, parent commit, and push remote to the implementer.

At coordinator handoff, stop all authors in that stack, detach their clean worktrees, and create one explicit detached coordinator at the recorded stack parent. Resume original authors against the coordinator path and selected branch one at a time; do not request new native worktree isolation for remediation.

Bind the installed `code_reviewer` to the assigned existing worktree and specify the exact `<base>..<head>` range for its bundled review. Spawn it from the current checkout if that is the assigned worktree, or pass the assigned path as `cwd` if `Agent` exposes it. Otherwise, use `EnterWorktree` only when the host permits entering that path, keep the checkout until the reviewer returns, then use `ExitWorktree` to restore the parent checkout and keep the worktree. If the host cannot bind the reviewer to the exact checkout, follow the role's incomplete review path.

This canonical skill is normally discovered in Claude through a `.claude/skills/epic-orchestrator` symlink. Select this adapter from runtime identity, not from that path.
