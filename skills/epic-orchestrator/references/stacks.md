# Manage a dependent PR stack

Before assigning stack work, verify Git 2.54 or later, the availability of `gh stack init`, `checkout`, `rebase`, and `sync`, and support for same-repository stacked PRs. Record the GitHub repository, remote, trunk, bottom base SHA, and approved branch order. Hold an affected stack when its prerequisites are unavailable.

In each coordinator, set `stack_repo` to the verified `host/owner/repo` and `stack_remote` to its remote. Prefix every `gh stack` invocation with `GH_REPO="$stack_repo"`, including initialization and readback. Before each push, verify that the effective fetch destination and every effective push destination identify that same repository.

## Transfer the stack to one coordinator

1. Wait for every layer's initial commit. Stop the writers for this stack and confirm all its issue worktrees are clean, including untracked files. Send dirty worktrees back to their authors. Record each branch, head SHA, and worktree path before detaching any issue worktree from its branch.
2. Leave the issue worktrees detached. Create one coordinator worktree detached at the recorded bottom base. If the local trunk is missing, create it at the published trunk SHA from the verified remote. Verify the recorded branch tips, then check out the bottom branch in the coordinator before running `gh stack init --base <trunk> <bottom-branch> ... <top-branch>` to adopt the layers in order. Check `gh stack view --json` before using an initialized coordinator, including on resume.
3. Use `gh stack checkout <branch>` in the coordinator before each layer assignment. Give only one writer access to that coordinator at a time and hold its checkout while a reviewer uses it. Keep other stacks and independent issues on their own worktrees.

## Review, publish, and link the stack

1. Reconcile locally with `gh stack rebase --remote "$stack_remote"`. Resolve the trunk target SHA reported by rebase and hold publication unless it matches the full SHA fetched for the published trunk. Run the global workflow's local review for each layer using that published SHA for the bottom layer and the preceding layer's exact head for each higher one. If a fix changes a lower layer, propagate it before reviewing the affected layers again.
2. Publish the exact reviewed heads in one atomic push, with explicit leases against the recorded remote heads for replaced refs. Create missing PRs as drafts bottom to top under the global PR policies. Target trunk for the bottom PR and the previous branch for each higher one. Check existing PRs against the approved order and preserve any ready state.
3. Run the required [guarded sync](#sync-published-layers) from the coordinator once the drafts exist. Verify the native stack before starting the bottom-to-top per-PR Libra workflow and applying the global readiness policy.

## Correct and propagate layers

Work bottom to top through the global workflow's per-PR findings batches. Resume the assigned coder on the coordinator with that layer checked out. After a lower-layer correction, propagate it locally with `gh stack rebase <branch> --upstack --no-trunk` when descendants exist. Run checks and complete local review for the corrected layer and each affected comparison. Publish their reviewed heads together in the batch's single push, then sync the stack again. Use the global workflow's current-head rule for each updated PR.

When a local rebase stops on a conflict, route the affected branch to its original coder or a replacement with that issue's approved scope and upstream contracts. Verify the staged resolution fits that scope before `gh stack rebase --continue`. Route any scope change through the epic's clarification and approval gate. Complete review of the resulting comparison before publishing.

While stack recovery is pending, defer native stack mutations in other coordinators in the same clone. Continue unrelated work that can proceed.

## Sync published layers

Before each sync, require `gh stack view --json` to report the intended PR association for every layer. In the verified repository, confirm each association's PR number, URL, head repository, branch, reviewed SHA, and expected base. Hold the stack if an association is missing or differs from the intended PR.

`gh stack sync` fetches and can rebase and push the stack. Publish the reviewed heads first, then run sync with a command-scoped hook that rejects ref updates while allowing a no-op push. It preserves existing hooks and catches changes that arrive between the local review and sync.

```sh
GH_REPO="$stack_repo" git \
  -c hook.epic-sync-guard.event=pre-push \
  -c 'hook.epic-sync-guard.command=f() { if IFS= read -r ref; then echo "Sync attempted a ref update" >&2; return 1; fi; }; f' \
  -c 'alias.epic-stack-sync=!f() { git hook list pre-push | grep -Fxq epic-sync-guard && exec gh stack sync "$@"; }; f' \
  epic-stack-sync --remote "$stack_remote"
```

Verify the local and published heads, PR bases and readiness, and native stack order after each sync. Use `gh stack view --json` and confirm the live PRs and stack membership on GitHub. Sync can report success after a rejected push. Treat a rejected push, unexpected reconciliation, or a failed readback as incomplete. Hold the affected stack for reconciliation and review before publishing any changed comparisons.
