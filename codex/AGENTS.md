# Plans

- When the user corrects a plan, or clarification establishes a concrete change to it, update the actual plan in the same turn. Do not stop at explaining what should change, proposing wording, or offering to apply a correction that is already clear and authorized.

# Global workflow

- The primary agent owns the user conversation, orchestration, and final handoff; specialists perform their assigned roles directly.
- Use each installed role definition and assign bounded ownership with the relevant requirements, revisions, and review context.
- Use fresh `plan_validator`, `code_reviewer`, and `finding_validator` instances with `fork_turns="none"`.
- Preserve the primary agent's settings; role definitions set specialist models and reasoning.
- For issue planning or Plan mode, use the planning flow; otherwise act on user changes and comments through `coder` and local review without a formal plan or extra approval. Answer questions directly.

# Task context

- Keep compact state in conversation: requirements or assignment, any plan version and approval, base and current revisions, correction counters, Libra applicability and deadline, accepted/rejected findings, and pending review IDs.
- Preserve this state through compaction and handoffs. Pass original reviews directly and unchanged; keep verification notes separate from the original JSON and findings.

# Global contract

- Choose the smallest scoped design that satisfies the requested behavior and applicable repository instructions.
- Trace existing capability owners, callers, and consumers through the relevant interfaces and trust boundaries.
- Search by responsibility across the repository, beyond changed or proposed files.
- Extend existing implementations and preserve authoritative representations instead of duplicating their responsibilities.
- Question redundant responsibilities and unnecessary structure in any form.
- Establish need from actual consumers and behavior; tests of accidental internal structure do not make it a requirement.
- Reuse appropriate fixtures and scenario tables when they cover the same contract.
- Distinguish similar code required by different behavior or boundaries from needless duplication.
- Support duplication claims with concrete evidence and a safe consolidation or deletion path; a replacement symbol or fewer lines is not proof.
- Keep work within the requested behavior and return material scope departures to the primary agent.

# Planning and approval

1. Clarify the requested outcome and delegate exploration and planning to `planner`.
2. Send the returned plan to a fresh `plan_validator`.
3. Route material objections to `planner`, then send its revision to a fresh `plan_validator`. Allow three correction rounds, each a planner revision followed by validation; ask the user before a fourth if material objections remain.
4. Present the planner's validated result as returned; do not author or reformat it. Use the required `<proposed_plan>` wrapper in Plan mode and obtain approval before implementing the plan. Plan-mode restrictions apply until the mode changes.
5. Route user plan corrections to `planner` and update the actual plan in the same turn. Material revisions return through fresh validation and approval; routine in-scope file assignments can change without reopening approval.

# Implementation and local review

1. Assign implementation to `coder`, then send the complete change and validation context to a fresh `code_reviewer`.
2. Pass the original review directly and unchanged to a fresh `finding_validator`.
3. Send all accepted original findings and verification notes to `coder` in one assignment and fix batch, then review and verify the complete batch. Preserve rejections with reasons and reconsider them only with new evidence.
4. Allow three local correction rounds per cycle. One round is a complete coder fix batch followed by review and finding validation. Each Libra fix batch resets only the local counter and starts a new local cycle at round one, leaving at most two additional local correction batches. Stop before a fourth round if accepted material findings remain; preserve the work and PR state for user intervention.
5. When no accepted material findings remain, perform final integration checks before publication.

# Draft PR and Libra

- After local review, create a draft PR under the PR policies below and attach its HTTPS URL to the task when supported.
- Libra is optional. Make a bounded applicability check using repository instructions, settings, required checks, and current or recent integration evidence. Use local review alone when optional Libra is explicitly disabled or none is detected. A missing status alone never disables a known integration; record applicability and evidence.
- When Libra applies, honor configured required labels, any draft-bypass label needed for eligibility, and review policies. Poll the `Libra` status for the exact current head SHA and inspect its matching review result and comments; a successful status can contain findings. Treat uncertain result matching as incomplete. Retain applicable older feedback and exclude superseded runs.
- Wait at most **10 minutes per head** from its first poll. Record the deadline in conversation and retain it across resumptions; use waits of at most 60 seconds and give concise progress updates.
- Validate findings from each matched current-head Libra review together. Send all accepted original findings and verification notes to `coder` in one assignment and fix batch, complete the batch's local review cycle, then push once and inspect the new head. Do not run a separate coder/review/push cycle per finding.
- Each Libra fix batch counts as one of three Libra correction rounds per PR and restarts the local cycle at round one. Reset only the local counter; preserve the Libra count across heads and ask the user before a fourth batch. Process later Libra results the same way without waiting for future runs to enlarge a batch. Leave thread replies and resolution to Libra.
- Mark the PR ready after completed local-only review or a settled Libra review with no accepted material findings. Verify the published state and latest applicable Libra activity before preparing focus comments.
- Preserve the draft with an explicit caveat when applicability is unresolved or expected Libra results fail, are unavailable, or remain incomplete at the deadline. Preserve the current PR state on correction-limit escalation.

# Pending focus comments and handoff

1. Identify meaningful areas for the user's judgment on the current diff. Skip areas the user has already reviewed or removed; do not recreate their focus comments.
2. Read the current head and the user's pending review. Reuse it or create one if absent, preserve all user comments, and add only missing focus comments with valid diff anchors. If the head changes, reconcile again before writing.
3. Use `gh api` with a JSON payload file; omit `event` when creating the review. Verify `PENDING` state, comment content, and current anchors. Never submit the review. Preserve existing drafts and report any API limitation.
4. Hand off the HTTPS PR link, validation results, review path and Libra outcome, material caveats, and pending focus-comment state. Keep focus guidance in the inline comments and report unavailable comments or checks honestly.

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
