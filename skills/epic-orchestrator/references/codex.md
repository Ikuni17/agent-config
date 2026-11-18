# Codex adapter

Use this adapter only when system instructions identify Codex or the available multi-agent tools are Codex collaboration tools.

## Mode and permissions

Run the full workflow in normal/default mode. Codex Plan Mode prohibits the scratch writes required by Phases 0-4; if active, stop and ask the user to switch modes. The workflow's Phase 4 approval gate still blocks implementation.

The orchestrator needs workspace write access for the scratch files. Planning and review agents return results without writing files. Verify repository status after every hard barrier.

## Agent operations

- Before Phase 0, determine the effective spawned-thread cap from host metadata and the applicable Codex configuration when available. `agents.max_concurrent_threads_per_session` excludes the primary thread. Count currently open subagent threads and record the configured cap, observed ceiling, occupied slots, and usable child capacity in `index.md`. Treat the lower host-enforced value as authoritative when configuration and observed capacity differ. Do not fill every slot merely to probe an otherwise undisclosed ceiling.
- Treat fewer than four usable child slots as low capacity. Notify the user before fan-out with the configured or observed cap, the currently usable count, and the expected queueing impact. Recommend `[agents] max_concurrent_threads_per_session = 16` plus a Codex restart when the configured cap is lower; continue with the bounded queue unless no child slot can become available.
- Spawn installed role agents with `spawn_agent`, the role name as `agent_type`, and stable issue-keyed `task_name` values. Use `fork_turns="none"` for the fresh independent roles specified in the Global workflow.
- Use the installed role definitions' model and reasoning settings.
- Record the canonical task name/ID returned for planning and implementation authors.
- Inspect live capacity with the available agent-list/status operation. Keep the primary orchestrator slot free and queue work beyond the current limit.
- Wait with `wait_agent`; do not busy-poll agent output.
- Resume an idle author with `followup_task`. Steer a running author with `send_message`.
- Spawn a new task name for every reviewer. Completed reviewer threads may be closed when the host permits, after their verdict is recorded.

Codex surfaces subagent threads to the user and returns their final messages to the parent. Store required role outputs in the scratch artifacts and keep compact status in the parent context.

## Worktrees

Codex child spawning does not itself guarantee a new worktree. For initial implementation, resolve and record the parent commit before creating a new branch:

```bash
git worktree add -b <branch> <absolute-worktree-path> <parent-commit>
```

For a resumed branch, inspect `git worktree list --porcelain`. Reuse its registered worktree when present; otherwise attach the existing branch without `-b`:

```bash
git worktree add <absolute-worktree-path> <branch>
```

Never reset or recreate an existing branch to force a parent match. Before a new implementation, verify that the worktree's `HEAD` equals the recorded parent commit. On resume before the coordinator handoff, verify the recorded parent commit is an ancestor of the branch. Pass the absolute worktree path, branch, parent commit, and push remote in the role prompt, and require all implementation commands to use that working directory.

At the shared workflow's coordinator barrier, stop every author in that stack before inspecting or detaching worktrees. Create the coordinator as an explicit detached worktree at the recorded stack parent. After handoff, resume the original author with the coordinator path and assigned branch; never let two authors use the same coordinator concurrently.
