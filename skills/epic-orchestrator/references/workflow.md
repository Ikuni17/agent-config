# Shared epic workflow

Use the selected host adapter for every operation named below. Apply the installed Global workflow for individual issue planning, implementation, review, and PR policies.

## Capability interface

The adapter must define how to:

- determine writable planning mode and the live child-agent capacity;
- spawn an installed role and record a stable handle when available;
- wait for agents and fill freed capacity from a queue;
- send feedback to a running author or resume a completed author;
- replace an unreachable author from filesystem artifacts;
- isolate an implementer in an exact Git worktree;
- stop stack authors and transfer their clean branches to one coordinator worktree;
- preserve returned role output in scratch artifacts and keep compact status in the orchestrator context.

Stop before Phase 0 if the adapter cannot provide scratch-file writes, independent reviewers, bounded scheduling, implementation isolation, or the coordinator handoff.

## Workspace and safeguards

Resolve absolute paths for the repository root, a new temporary scratch directory outside the repository, and every implementation worktree. Key directories, task names, state, and user references by issue number:

```text
<scratch>/
  index.md
  review.md
  worktrees/
    stack-1-coordinator/
  57-reviewer-interface/
    exploration.md
    plan.md
    notes.md
    clarification.md
    review.md
    pr-review.md
```

Record in `index.md`:

- repository, epic, initial repository status, push remote, default branch, and default remote-tracking ref;
- dependency graph, planning waves, PR boundaries, and global stack/merge order;
- live capacity plus queued and running counts;
- per-issue status, summary, artifact paths, approved scope, requirement and contract traceability, plan verdict, and review iterations;
- per-issue clarification state, exact question, proposed disposition, and verbatim developer response;
- author handles or replacement-author state;
- branch, stack parent branch and commit, initial worktree path, commit, and checks;
- per-stack coordinator path, handoff state, and branch SHAs before and after each rebase or sync;
- per-PR issue/epic linkage, URL, base, current and ever-ready state, author, assignees, independent-review history, Libra result, and unresolved-thread count.

Before spawning agents, capture `git status --porcelain=v1 --untracked-files=all`. After every exploration, planning, review, and revision wave, compare the same output byte-for-byte with the baseline. If it differs, stop spawning agents, report the paths, and ask the user how to handle the unexpected repository writes. Never discard them.

The orchestrator writes the scratch artifacts from the installed roles' returned results. Preserve accepted plans and original reviews unchanged, with verification notes separate. Treat the repository as read-only during Phases 0-4 and keep the Global workflow's compact task state per issue and PR.

## Scope and clarification barrier

The allowed scope for an issue is the smallest changeset traceable to its issue text and the explicit contracts it must provide to or consume from other issues in the epic. A contract does not silently expand an issue: when the issue text is insufficient to authorize the work required by that contract, clarification is mandatory.

Repository rules remain acceptance criteria but do not expand issue scope. If complying with a rule requires an untraceable change, apply this barrier before making it.

At any phase, if an agent identifies an untraceable change, missing requirement, or incompatible contract:

1. Stop the affected issue before adding the proposed work to its plan or implementation.
2. Ensure `clarification.md` records the exact gap, the requirement or contract at risk, and the proposed disposition: remove the untraceable work or approve the smallest necessary scope addition.
3. Mark the issue `blocked — developer clarification` in `index.md`, remove it and its dependents from ready queues, and present the clarification to the developer.
4. Record the developer's response verbatim. Only explicit approval of the stated addition expands the issue scope; epic execution approval and approval of a different addition do not.
5. Resume the original author or replacement fallback with the recorded response. A direction to remove or reject unapproved work permits only that exact disposition; independently re-review any changed plan or diff. If the developer approves a scope addition, complete plan revision and fresh independent plan review before implementation begins or resumes.

If approval is withheld, keep the issue blocked unless the developer explicitly directs removal or rejection of the unapproved work. Never infer a disposition, broaden an addition, or continue dependent planning from an unresolved contract.

## Inputs and preflight

Require an epic number or URL and resolve `owner/name`. Default to one draft PR per issue, stacked when dependencies exist.

Before Phase 0:

1. Verify `gh auth status`.
2. Verify `github/gh-stack` appears in `gh extension list` and `gh stack init --help`, `gh stack rebase --help`, `gh stack sync --help`, `gh stack checkout --help`, and `gh stack view --help` succeed. If any capability is missing, stop and identify the dependency; do not silently replace the stack workflow.
3. Resolve the repository root, push remote, and default branch without changing branches. Record the remote explicitly; do not rely on `gh stack` auto-detection when multiple remotes exist.
4. Create the scratch workspace and initialize `index.md`.
5. Ask the adapter for live child capacity and initialize the bounded queue.

Use the installed role definitions' model and reasoning settings.

## Phase 0 — Map the epic

Use `gh` to fetch the epic, enumerate sub-issues, read every body, and find already completed work:

```bash
gh issue view <epic> --repo <owner/name> --json number,title,body,url,state
gh api repos/<owner/name>/issues/<epic>/sub_issues \
  --jq '.[] | {number, title, state, type: .type.name}'
gh issue view <issue> --repo <owner/name> --json title,body --jq '.title, "---", .body'
```

If there are no GitHub sub-issues, parse the epic task list. Read relevant merged PRs and scope every plan to remaining work.

Build dependencies from consumed types, interfaces, endpoints, migrations, and other cross-issue contracts. Reject cycles. For every issue, record which issue requirement authorizes each contract obligation. Apply the scope and clarification barrier when the issue text does not authorize required contract work. Record the graph, topological planning waves, and proposed PR stack order.

## Bounded scheduling

Use issue-keyed names such as `issue_57_explore`, `issue_57_plan`, `issue_57_plan_review_1`, `issue_57_implement`, `issue_57_rebase_conflict`, and `issue_57_pr_review_1`.

For every phase or dependency wave:

1. Queue all ready units in `index.md`.
2. Spawn only up to the adapter's available child capacity.
3. Record each author handle before other work when the host supplies one.
4. Wait for completions, update state, and fill freed slots.
5. Treat capacity errors as queue signals, not issue failures.
6. Enforce a hard barrier: do not start the next phase or wave until the current queue is empty, every agent is finished, `index.md` is current, and the repository status safeguard passes.

An issue blocked on developer clarification is not ready. Its dependents remain blocked until its approved contract is available.

Always spawn a new uniquely named agent for a review or re-review. For revisions, use the adapter's original-author continuation. If it is unavailable, spawn a replacement author in the installed role with the issue's exploration, plan, notes, review, and upstream contracts before revising the work.

## Phase 1 — Explore

Assign the installed `planner` one exploration unit per coherent code area. Require actual types, call chains, tools, tests, serialization boundaries, reuse opportunities, and exact `file:line` evidence. Ask for investigation only and record its evidence in `exploration.md`. If its evidence shows that the issue text cannot authorize a required epic contract, apply the scope and clarification barrier before planning.

## Phase 2 — Plan in dependency waves

Plan an issue only after every upstream contract is available:

1. Plan unblocked issues with no unplanned dependency.
2. Run ready issues through the bounded scheduler.
3. Require every `plan.md` to contain `## Scope`, mapping each planned change and file to an issue requirement, an issue-authorized epic contract, or a recorded scope approval, to state the exported contract downstream issues use, and to contain a `## Pull requests` section declaring one PR or an approved split, ordered PR boundaries, and dependencies.
4. Give later waves absolute paths to upstream plans rather than relaying their text.
5. Repeat until all issues are planned.

Use the installed `planner` role's plan format. Put alternatives and derivations in `notes.md`, not the presentable plan. A planner that needs an unapproved scope addition reports the gap separately from the plan. Record it in `clarification.md` and apply the scope and clarification barrier before continuing that wave.

## Phase 3 — Adversarial plan review

Use a fresh `plan_validator` for every plan under the Global workflow. Give it the plan, the planner's separate investigation handoff, and the exploration artifacts. Ask it to check requirement and contract traceability, PR boundaries, and cross-plan consistency. Save its returned review in the issue's `review.md`.

Route in-scope findings to the author through the adapter. Mark the issue `revised — re-review pending`, then spawn a fresh `plan_validator` with an incremented iteration. Apply the scope and clarification barrier when the plan contains untraceable work, satisfying a finding requires unapproved scope, or the issue text cannot satisfy an epic contract. Use the Global workflow's plan disposition and correction limits.

## Phase 4 — Assemble and stop

Do not assemble while any issue or contract is blocked on clarification. Write `<scratch>/review.md` with a bounded summary per issue: scope, approach, files, decisions, dependencies, PR boundaries, contract, verdict, and an absolute link to the full plan. Add only an epic header, dependency graph, and global PR stack/merge order. Designate exactly one final PR to close the epic; it must also be the final PR for its issue.

Present the review document with the accepted plans unchanged and stop. Do not fetch, branch, create worktrees, implement, push, or open PRs until the user explicitly approves execution.

Route user revisions through the same author and independent re-review loop, then regenerate the affected summary. Execution approval authorizes only implementation of scopes that received any required separate scope approval; it never approves a scope addition.

## Phase 5 — Execute after approval

After explicit approval:

1. Fetch the latest default branch without overwriting local work: `git fetch <remote> <default-branch>`. Resolve `<remote>/<default-branch>` to a commit and record it as the bottom stack parent.
2. Process initial implementation in dependency waves. Create a dependent worktree after its parent's initial implementation commit exists, record that exact parent, and continue other ready work in parallel. Independent stacks may also proceed concurrently.
3. Ask the adapter to create or resume each issue branch and worktree from its recorded parent. Verify the absolute path, checked-out branch, and ancestry before spawning the implementation agent. Require every repository command and edit to run inside that worktree.
4. Give each `coder` the verbatim issue text, approved plan, contract paths, recorded scope approvals, and clarification dispositions. Have it apply only changes traceable to the plan's `## Scope` or exact developer-directed removals, treat repository rules as acceptance criteria, audit every comment visible in the parent-to-HEAD diff, run project checks, and commit with project conventions. Complete the Global workflow's local review before authorizing it to push only its assigned branch with `git push --set-upstream <remote> <branch>`. Record the commit and checks.

If implementation discovers necessary work outside the approved scope, the `coder` must stop before making that additional change, preserve existing in-scope work, and report the gap. Record it in `clarification.md` and apply the scope and clarification barrier. If an addition is approved, revise and independently re-review the plan before resuming implementation; if removal is directed, make only that exact removal and re-review the resulting diff.

Throughout Phase 5, refresh `isDraft` before every PR mutation or stack synchronization. A PR that is ready for review has been intentionally advanced by the developer. Never invoke `gh pr ready --undo` or any equivalent operation, and use only mutations that preserve its ready state. Verify afterward that every PR which was ready remains ready.

For every rebase conflict, stop automatic reconciliation and resume the `coder` responsible for each conflicting branch with a bounded conflict-resolution assignment, approved plan, and recorded scope decisions. Require the smallest resolution that preserves the approved diff, verify the complete staged diff and worktree status against `## Scope`, and apply the scope and clarification barrier before any untraceable resolution. Continue the rebase only after the responsible author reports a scope-clean resolution; repeat for each subsequent conflict.

### Coordinator handoff

After every branch in one stack has an initial implementation commit, stop all authors in that stack and establish a hard barrier:

1. Require empty output from `git -C <issue-worktree> status --porcelain=v1 --untracked-files=all` for every issue worktree. If any worktree is dirty, stop and route it to its author; never discard changes.
2. Record each current branch and SHA with `git -C <issue-worktree> branch --show-current` and `git -C <issue-worktree> rev-parse HEAD`, then detach it with `git -C <issue-worktree> switch --detach`.
3. Create `<stack-coordinator>` detached at the recorded bottom stack parent, never at an attached local default branch:

   ```bash
   git worktree add --detach <stack-coordinator> <bottom-parent-commit>
   git -C <stack-coordinator> switch <bottom-branch>
   ```

4. From the coordinator, adopt branches in bottom-to-top order and reconcile the stack. Run these commands separately and complete global local review for any changed comparison after rebase and before publishing with sync:

   ```bash
   gh stack init --base <default-branch> <bottom-branch> ... <top-branch>
   gh stack rebase --remote <remote>
   gh stack sync --remote <remote>
   ```

5. Verify `gh stack view --json` and `git merge-base --is-ancestor <lower-branch> <upper-branch>` for every adjacent pair. Record every resulting branch SHA. Do not assume `gh stack init` leaves a particular branch checked out.

Leave the issue worktrees detached. The coordinator becomes that stack's only active implementation worktree. Resume original authors or the adapter fallback there one at a time, always selecting the assigned branch first with `gh stack checkout <branch>`.

### PR creation

Create every new pull request as a draft explicitly after the first coordinator sync. The first two physical body lines must be:

```text
<Closes|Part of> #<issue>
<Part of|Closes> #<epic>
```

Use the Global workflow's PR creation rules for issue references and metadata. Use `Closes #<epic>` only on the single final PR in the global merge order. Put all other description content after those lines. Create from a scratch body file to avoid quoting errors:

```bash
gh pr create --repo <owner/name> --draft --base <base-branch> --head <branch> \
  --title "<title>" --body-file <absolute-body-file> --assignee Ikuni17
```

Use the default branch as the bottom PR base and the preceding stack branch as each higher base. Fetch each created PR's `author`, `assignees`, `baseRefName`, `body`, and `isDraft`; record whether it has ever been ready for review. If its author is not assigned, run `gh pr edit <number> --repo <owner/name> --add-assignee <author-login>`. Verify the assignment and exact first two body lines. After every PR exists, run `gh stack sync --remote <remote>` from the coordinator and verify the PR bases, stack, and draft states again.

### Independent PR review

Process each stack bottom-to-top. Follow the Global workflow's local `code_reviewer` and `finding_validator` cycle before publishing each branch or changing a published comparison, including when sync publishes rebased layers. For each comparison, give the fresh reviewer the assigned worktree, exact base and head, approved scope and plan, acceptance criteria, recorded approvals, repository rules, and every comment visible in changed hunks. Include `gh pr diff` for an existing PR when it matches that comparison. Preserve the returned review in `pr-review.md`. Apply the scope and clarification barrier to every diff item that is not traceable to approved scope; the developer decides whether to remove it or approve the smallest necessary scope addition. Route accepted in-scope findings to the original `coder` or adapter fallback and complete local review before syncing the stack. If sync changes the reviewed comparison, re-review the resulting revisions before advancing. If a scope addition is approved, revise and independently re-review the plan before resuming implementation review. Preserve every review iteration in `index.md` and enter feedback remediation only after local review is complete.

### GitHub feedback remediation

Libra starts from GitHub pull-request events; never trigger it directly. Resume the original `coder` or adapter fallback with the accepted findings, approved scope, and coordinator path.

For each PR in bottom-to-top order:

1. Use the Global workflow's Libra applicability, exact-head result and deadline checks, findings validation, and correction batches. Follow its rules for replies and Libra resolutions.
2. Fetch every unresolved GitHub review thread, regardless of author. Apply the following thread handling to human feedback.
3. For a thread requiring code within the approved scope, implement and check the fix on the assigned coordinator branch, commit it, and push only that branch after global local review:

   ```bash
   git push <remote> <branch>
   ```

   Resolve the thread only after the fixing commit is pushed. Do not rebase or sync the stack while this PR still has unresolved feedback.
4. For a thread not requiring code, post a concise explanation and resolve it.
5. After a code-changing push, apply the Global workflow to Libra's next result when it applies and repeat until the feedback is settled.
6. When remediation for this PR ends, propagate its stabilized commits once before advancing to the next PR or handing off to the human. Run these commands separately and complete global local review for changed comparisons after rebase and before publishing with sync:

   ```bash
   gh stack rebase <branch> --upstack --no-trunk
   gh stack sync --remote <remote>
   ```

   Omit the rebase when the branch has no descendants. Record the resulting stack SHAs and the actual unresolved-thread count.

If a thread requests work outside the approved scope, pause before replying, resolving, or editing and apply the scope and clarification barrier. If the developer rejects the request, reply only as directed and resolve it. If the developer approves a scope addition, revise and independently re-review the plan before resuming remediation.

On a rebase conflict, apply the Phase 5 rebase-conflict procedure, then run `gh stack rebase --continue` and `gh stack sync --remote <remote>` with global local review before sync publishes changed comparisons. Follow the Global workflow for readiness, caveats, and pending focus comments.

Never merge. Report merge order and preserve each PR's current draft or ready-for-review state. For each PR report its URL, independent-review verdict and history, Libra result, unresolved-thread count, and any visible terminal failure requiring human judgment.

If the dependency graph cannot be represented by the existing stack policy, report the ambiguity at Phase 4. Do not invent a new DAG linearization during execution.
