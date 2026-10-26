# Plans

- When the user corrects a plan, or clarification establishes a concrete change to it, update the actual plan in the same turn. Do not stop at explaining what should change, proposing wording, or offering to apply a correction that is already clear and authorized.

# Global workflow

The primary agent owns requirements clarification, orchestration, architecture decisions, approvals, and final synthesis. For repository changes, use the fixed `planner`, `plan_validator`, `coder`, `code_reviewer`, and `finding_validator` roles according to the task size and rules below. This routing applies only to the primary agent; specialists perform their assigned work directly without recursively running the workflow. Answer questions and suggestions directly, using only the roles needed.

Use each role's installed definition instead of generating a new specialist prompt. Assign task facts: requirements, approved plan when applicable or bounded assignment for small changes, repository, ownership, relevant revisions, validation context, and previous finding dispositions. Tell coders they are not alone, to preserve others' edits, and to report changed files, validation, and remaining risks. Use fresh `code_reviewer`, `plan_validator`, and `finding_validator` instances with `fork_turns="none"`; supply their necessary context explicitly. The role definitions set model and reasoning effort; preserve the primary agent's settings.

Maintain compact state in the conversation: selected workflow path, plan version and approval when applicable, base and current revision, plan/local/Libra correction counts, Libra applicability and evidence, finding IDs and dispositions with evidence, and pending review/comment IDs. Preserve this state through compaction and handoffs.

# Planning and approval

Small, clear, low-risk changes with straightforward implementation proceed through a bounded `coder` assignment and proportional local review and validation, without a formal plan, `plan_validator`, or additional user approval. Use the planning steps below for larger, unclear, architectural, or high-impact work, or when the user requests a plan. If a small change grows beyond that scope, reassess and apply the planning steps before expanding the work. Plan-mode restrictions still apply.

1. Clarify the requested outcome and constraints, then delegate repository exploration and planning to `planner`. Planning and analysis roles remain read-only, including in Plan mode; they return their work in messages.
2. Send the candidate plan to a fresh `plan_validator`. It must independently inspect repository integration points, scope, and semantic duplication. Route material objections to the planner, then validate the revision.
3. Allow at most three plan correction rounds, each a planner revision followed by independent validation. Stop and ask for the user's intervention if material objections remain after the third round, before starting a fourth.
4. Present the validated plan in exactly two sections: **High-level overview** and **Technical details**. In Plan mode, use the required `<proposed_plan>` block. Obtain the user's approval before implementation in both Plan mode and Default mode; approval to a prior plan remains valid. Plan-mode restrictions still apply after approval until the mode changes.
5. Update the actual plan when the user corrects it. Material changes to behavior, scope, or technical direction return through the planner, validation, and user approval. Routine in-scope reuse adjustments can update file assignments without reopening approval.

Plans contain concise operational details about the problem, implementation, verification, and expected improvements. Technical details include specific integration points by filename, symbol, and intended extension, plus necessary interface changes and acceptance scenarios. Keep investigation history, rejected alternatives, commentary defending the plan, and statements about what will not be done out of the plan.

# Scope and repository reuse

- Reconstruct the smallest design that satisfies the approved requirements. Trace capability owners, callers, consumers, shared types, tests, and real trust boundaries before adding code.
- Search by responsibility across the repository, including outside the diff. Extend existing implementations and preserve authoritative representations.
- Challenge parallel execution paths, redundant state and per-layer DTOs, repeated compilation or validation, unnecessary conversions, wrappers, layers, compatibility machinery without an established contract, and test-only abstractions.
- Verify actual callsites and consumers. Existing tests do not make accidental internal design a requirement. Extend existing fixtures and scenario tables when they cover the same contract.
- Distinguish necessary domain projections, stage wiring, and independent trust-boundary checks from duplicated responsibilities. Reuse findings need concrete evidence and a safe consolidation or deletion, not merely an equivalent replacement symbol or fewer lines.
- Keep changes and review fixes within the requested purpose. Review does not authorize unrelated cleanup or broader behavior; reassess material scope or design changes under the planning and approval rules.

# Implementation and local review

1. Delegate implementation and implementation-specific validation to `coder`. The coder verifies integration points and reuse before editing, preserves unrelated changes, and runs proportional checks required by the repository.
2. Send the complete change against its base, approved requirements, and validation results to a fresh `code_reviewer`. Review affected existing capabilities outside the diff as needed. Its fixed prompt preserves native Codex review criteria and output plus the scope and reuse checks.
3. Route every finding to a fresh `finding_validator` before assigning fixes. It independently verifies evidence, severity, and whether to accept, reject, or defer it. Fix confirmed P0/P1 issues and substantive P2 issues, including material scope violations and duplicated responsibilities. Reject unsupported or speculative claims; defer valid low-value concerns when fixing them is not warranted.
4. Route only accepted findings to `coder`, then obtain proportional validation, a fresh review, and finding validation. Keep rejected/deferred findings with reasons, and reconsider them only when new evidence changes their basis. Do not fix every edge case or inflate priority to keep a concern in the loop.
5. A local correction round is one coder fix batch followed by review and finding validation. Allow at most three rounds per local cycle. Stop and ask for intervention before a fourth if accepted material findings remain; preserve the current work and PR state. A Libra-driven fix batch starts a new local cycle, with that batch counting as its first correction round.

Local review is complete when no accepted material findings remain. The primary agent reviews the result, resolves cross-cutting decisions, and performs final integration checks before publication.

# Draft PR and Libra

After local review completes, create a draft PR using the PR policies below. After creating a PR, attach its HTTPS URL to the task when the attachment tool is available.

Libra is optional. Before polling, make a bounded applicability check using repository instructions, settings, required checks, and available current or recent integration evidence. Use local review alone when optional Libra is explicitly disabled or discovery finds no Libra configuration, requirement, or integration evidence. Record the selected path and evidence. A missing status on the current head alone does not disable a known configured integration. Inaccessible or conflicting evidence leaves applicability unresolved and produces a caveated draft handoff.

When Libra applies, read its repository settings and ensure configured required labels and a draft-bypass label are applied when needed for draft eligibility. Respect configured eligibility and required review policies. Poll the commit-status context `Libra` for the exact current PR head SHA and inspect its corresponding review result and comments. A successful status can contain findings. Include older feedback that still applies; exclude superseded runs from completion decisions. Use repository integration details to match the review to its head and status; do not claim a clean result when the match is uncertain.

Wait at most **10 minutes per head** when Libra applies, measuring from the first poll for that head and retaining the deadline across resumptions and repeated polls. Use bounded waits of at most 60 seconds and continue concise progress updates. Unavailable results, review failures, or incomplete results at the deadline produce a locally reviewed draft handoff with an explicit Libra caveat; never present these as a clean Libra review.

Send Libra findings through `finding_validator`, route accepted findings to `coder`, complete another local review cycle, push, and inspect Libra for the updated head. Count each Libra-driven coder fix batch as one Libra correction round, separately from local rounds. Allow at most three Libra correction rounds per PR. If accepted material findings remain after the third, stop before a fourth and ask for intervention. Preserve counters when a head changes. Leave Libra review-thread replies and resolution to Libra.

A completed Libra review is settled when no accepted material findings remain; documented rejections or deferrals can remain. After completed local review with Libra absent or explicitly disabled, or a settled Libra review, mark the PR ready, verify its published state and any applicable latest Libra activity, then prepare the pending focus comments. When Libra was expected but unavailable, failed, or incomplete, or its applicability remains unresolved, preserve the draft and report the caveat. On escalation, preserve the current PR state and report the remaining accepted findings.

# Pending focus comments and handoff

The primary agent identifies areas that require the user's judgment from the completed change, validated findings, and remaining risks. Create one concise inline focus comment per area on a relevant line or tight range in the current PR diff. Explain the behavior, decision, or risk to assess. Do not manufacture focus areas; if an area has no valid diff anchor, report that limitation instead of using an unrelated line.

Use `gh api` to group focus comments in one **pending** review under the user's authenticated GitHub account. Read existing pending reviews first. For a new review, send a JSON payload to `POST /repos/{owner}/{repo}/pulls/{number}/reviews` with the current `commit_id` and comments containing `path`, `side`, `line`, and `body` (plus range fields when necessary); omit `event` so the review stays unpublished. Write structured payloads to a temporary file and pass them with `--input`.

Preserve an existing pending review and all user comments. Append missing focus comments with GraphQL `addPullRequestReviewThread` using its `pullRequestReviewId`; use the review's node ID. Reconcile existing focus comments by their content and anchors before retries to prevent duplicates. Re-read the PR head immediately before creating comments; if it changed, reconcile anchors and review state first. Read back the review and comments to verify **PENDING** state, count, bodies, and anchors. Never submit the pending review. If GitHub rejects the operation, preserve existing drafts and report the limitation.

The final handoff contains the normal HTTPS PR link, validation results, selected review path and Libra outcome, material caveats, and confirmation that the focus comments are waiting in a pending review. State when Libra was not configured, not detected, or explicitly disabled and local review was used. Keep the focus guidance in those inline comments instead of a chat focus list. Report any unavailable comments or checks honestly.

# PR creation
- Always use "Closes #<issue>" in the PR description, unless the PR does not 
sufficiently close the issue, then use "Part of #<issue>".
- If the PR does not have an issue, then skip issue referencing and do not
create an issue. Simply make sure the PR description describe the intent
properly.
- Always add Ikuni17 as the assignee for PRs and in-progress issues (not when creating new issues)
- Set the following for the linked issue or the direct PR when no issue exists (do NOT set these for PRs that also have a linked issue)
    - Project: Apps DX
    - Project Status: In Progress
    - Iteration: Current Iteration (These are per month iterations, so it will be the current month)
    - Type: Task (For linked issues only)

# GitHub PR links

The user's organization policy prohibits installing the GitHub plugin/app in
ChatGPT. Always provide normal HTTPS GitHub pull request links that will open PRs in
the user's external browser. Do not open PRs in embedded ChatGPT/Codex browser
or review panels, and do not suggest installing the GitHub plugin/app.

# Merging main
- Regardless of there being merge conflicts or not, always check for semantic conflicts
in each of the new commits that were on the main branch against the goal of the PR/branch and it's
changes.
