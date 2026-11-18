---
name: epic-orchestrator
description: Coordinate planning and implementation across a GitHub epic or tracking issue in Codex or Claude Code. Use when the user wants linked issues planned or built together. Schedule independent work in parallel and use stacked pull requests for dependent work.
---

# Epic orchestrator

Run the installed global workflow in `AGENTS.md` or `CLAUDE.md` for each issue. The primary agent coordinates dependencies, approval, and handoff. Delegate to the installed `planner`, `plan_validator`, `coder`, `code_reviewer`, and `finding_validator` roles. In Codex, specify the named `agent_type` and use `fork_turns="none"` for each fresh independent review role. In Claude Code, call `Agent` with the exact `subagent_type` and a non-forked context.

Keep the global workflow's state in the primary conversation, keyed by issue and PR. Include dependency contracts, stack order, worktree paths, and reviewed revisions. Pass each role its bounded assignment, repository rules, approved dependencies, and the original role outputs required by the global workflow. Queue eligible work up to host capacity. Resume authors where supported, and keep unrelated issues moving when an issue and its dependents are blocked.

## Plan the epic

1. Resolve the epic number or URL to one GitHub host, repository, and default branch. Verify the local checkout's effective fetch remote against the epic repository and bind planners to that absolute checkout and the verified default-branch revision. Scope GitHub reads and PR writes to the resolved host and repository. Use `gh issue view` and the paginated `repos/<owner>/<repo>/issues/<number>/sub_issues` endpoint. Use the epic's issue checklist when it has no sub-issues. Read issue bodies, check status and merged work, and plan the remaining scope.
2. Map the dependencies and the contract each issue supplies or consumes. Give each downstream planner the validated upstream contracts. Default to one PR per issue in the epic repository. Surface proposed splits, cycles, ambiguous stack order, and contracts requiring added scope before approval. Pause affected issues and their dependents until the developer resolves the gap. Submit changed plans and affected downstream plans to fresh validation.
3. Present the dependency and merge order with the unchanged, independently validated plans for one approval using the host's global approval mechanism. In Codex Default mode, wait for explicit developer approval. Keep the repository and GitHub read-only until approval. Apply the global workflow's scope and approval rules again if implementation needs additional scope.

## Execute approved work

Record explicit fetch and push remotes. Verify the effective fetch URL with `git remote get-url <fetch-remote>` and every effective push URL with `git remote get-url --push --all <push-remote>`. Match them to the epic repository and the intended head repository respectively. Use the epic repository as the default head repository and require it for stacks. Use the verified remote names for each fetch and push.

Fetch and record the default branch head as the base for independent issues and bottom stack layers. Create a separate Git worktree and branch for each issue at its recorded base SHA. Verify the worktree, branch, and HEAD before assigning a coder. Pass each reviewer the absolute worktree path and exact base and head SHAs. Apply the global workflow's local review before publishing each draft. Coordinate Libra review, correction budgets, PR metadata, and pending focus comments per PR through that workflow.

On Claude Code, preflight whether `EnterWorktree` and `ExitWorktree` can switch to the assigned checkout and back. Enter the existing issue worktree and verify its root and head. Launch the fresh named `code_reviewer` and tell it to pass `high --max-findings all <base>..<head>` to its bundled review skill. Wait for that review before restoring the original checkout and keep the issue worktree. Serialize these checkout switches and keep other assignments pinned to their absolute paths. Hold the affected review and publication if the host cannot bind the reviewer.

## Publish dependent stacks

Preflight `gh stack link`, repository stack support, and writable head branches in the same repository before starting dependent work. Hold the affected stack when a prerequisite is unavailable. Process each stack bottom to top while independent issues and stacks proceed concurrently. Once the parent's review and corrections settle on a verified published head, use that SHA as the child's implementation base and the parent's branch as the child's PR base.

As soon as a locally reviewed child draft exists, link and verify the published prefix before waiting for that child's Libra result. Match each existing full PR URL's host, owner, repository, and number to the recorded stack. Require each new member to be open and eligible for linkage, with neither a queued merge nor auto-merge enabled. Pass the URLs in bottom-to-top order.

```sh
GH_REPO=<host>/<owner>/<repo> gh stack link --base <default-branch> <full-pr-urls-in-order>
```

Verify repository, base refs, head SHAs, and draft states before and after linkage. Confirm ordered membership with `gh api --hostname <host> 'repos/<owner>/<repo>/stacks?pull_request=<number>'`. Keep new stack PRs in draft until all planned members are linked and verified, then apply the global readiness rules. Preserve any ready state set by the developer.

Recheck live refs, reviewed comparisons, and expected membership for the affected PR and its published ancestors before advancing a child, marking ready, adding pending focus comments, or handing off. For a PR targeting the default branch, compare its live effective merge base with the recorded review base. A trunk advance with the same head and merge base preserves the review. Report any GitHub rebase requirement separately. For a child layer, require the expected parent SHA and ancestry. Reconcile changed comparisons before publication and repeat checks and local review. If a published comparison or membership changes unexpectedly, or a parent becomes ineligible before linkage, preserve the affected PR's published state and hold its descendants while it is unsettled. Report the blocker and continue unrelated work.

Apply the global issue reference rules and use `Part of #<epic>` to reference the epic. Use `Closes #<epic>` instead only on the designated final PR in an approved merge order that completes the epic. Flag issue and epic closure for verification when a PR targets a stack branch. Report the merge order and each PR's global workflow handoff.
