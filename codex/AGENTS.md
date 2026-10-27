# Global workflow

- The primary agent owns the user conversation, orchestration, and final handoff; specialists perform their assigned roles directly.
- Use each installed role definition and assign bounded ownership with the relevant requirements, revisions, and review context.
- A role whose skill is unavailable reports a blocker and stops.
- Use fresh `plan_validator`, `code_reviewer`, and `finding_validator` instances that inherit no conversation history (see Tool mechanics).
- Preserve the primary agent's settings; role definitions set specialist models and reasoning.
- For issue planning or Plan mode, use the planning flow; otherwise act on user changes and comments through `coder` and local review without a formal plan or extra approval. Answer questions directly.

## Task context

- Keep compact state in conversation: requirements or assignment, any plan version and approval, base and current revisions, correction counters, Libra applicability and deadline, accepted/rejected findings, and pending review IDs.
- Preserve this state through compaction and handoffs. Pass original reviews directly and unchanged; keep verification notes separate from the original review output and findings.

## Global contract

- Choose the smallest scoped design that satisfies the requested behavior and applicable repository instructions.
- Trace existing capability owners, callers, and consumers through the relevant interfaces and trust boundaries.
- Search by responsibility across the repository, beyond changed or proposed files.
- Extend existing implementations and preserve authoritative representations instead of duplicating their responsibilities.
- Before introducing a material identity, representation, lifecycle, or abstraction, identify the existing capability that owns the responsibility and evaluate extending, propagating, retaining, or deriving its representation.
- An existing mechanism being unsuitable as-is does not establish that a new mechanism is necessary. Require a concrete required scenario where the simpler extension fails before introducing a parallel responsibility.
- Question redundant responsibilities and unnecessary structure in any form.
- Establish need from actual consumers and behavior; tests of accidental internal structure do not make it a requirement.
- Reuse appropriate fixtures and scenario tables when they cover the same contract.
- Distinguish similar code required by different behavior or boundaries from needless duplication.
- Support duplication claims with concrete evidence and a safe consolidation or deletion path; a replacement symbol or fewer lines is not proof.
- Treat a material avoidable responsibility as a substantive scope or reuse finding when repository evidence supports a bounded simpler solution with equivalent required behavior. Identify the existing owner, the simpler extension, the added consistency, lifecycle, or maintenance burden, and a safe correction.
- Apply this finding standard during both review and adjudication. A runtime failure is not required; passing tests do not make a supported finding cosmetic.
- Keep work within the requested behavior and return material scope departures to the primary agent.

## Planning and approval

1. Clarify the requested outcome and delegate exploration and planning to `planner`.
2. Send the returned plan, original requirements, and the planner's separate investigation handoff for material additions to a fresh `plan_validator`.
3. Route material objections to `planner`, then send its revision to a fresh `plan_validator`. Allow three correction rounds, each a planner revision followed by validation; ask the user before a fourth if material objections remain.
4. Check the validator's independent decision evidence for every material addition before accepting validation and presenting the planner's result as returned; do not author or reformat it. In Plan mode, present it through the plan mechanism under Tool mechanics and obtain approval before implementing the plan. Plan-mode restrictions apply until the mode changes.
5. Route user plan corrections to `planner` and update the actual plan in the same turn. Material revisions return through fresh validation and approval; routine in-scope file assignments can change without reopening approval.

## Implementation and local review

The primary agent may skip `code_reviewer` and `finding_validator` for changes it judges simple. Run appropriate checks and disclose the skipped review and reason in the final handoff.

1. Assign implementation to `coder`, then send the complete change and validation context to a fresh `code_reviewer`.
2. Pass the original review directly and unchanged to a fresh `finding_validator`.
3. Send all accepted original findings and verification notes to `coder` in one assignment and fix batch, then review and verify the complete batch. Preserve rejections with reasons and reconsider them only with new evidence.
4. Allow three local correction rounds per cycle. One round is a complete coder fix batch followed by review and finding validation. Each Libra fix batch resets only the local counter and starts a new local cycle at round one, leaving at most two additional local correction batches. Stop before a fourth round if accepted material findings remain; preserve the work and PR state for user intervention.
5. When no accepted material findings remain, perform final integration checks before publication.

## Draft PR and Libra

- After local review, create a draft PR under the PR policies below and attach its HTTPS URL to the task when supported.
- Libra is optional. Make a bounded applicability check using repository instructions, settings, required checks, and current or recent integration evidence. Use local review alone when optional Libra is explicitly disabled or none is detected. A missing status alone never disables a known integration; record applicability and evidence.
- When Libra applies, honor configured required labels, any draft-bypass label needed for eligibility, and review policies. Poll the `Libra` status for the exact current head SHA and inspect its matching review result and comments; a successful status can contain findings. Treat uncertain result matching as incomplete. Retain applicable older feedback and exclude superseded runs.
- Wait at most **10 minutes per head** from its first poll. Record the deadline in conversation and retain it across resumptions; use waits of at most 60 seconds and give concise progress updates.
- Validate findings from each matched current-head Libra review together. Send all accepted original findings and verification notes to `coder` in one assignment and fix batch, complete the batch's local review cycle, then push once and inspect the new head. Do not run a separate coder/review/push cycle per finding.
- Each Libra fix batch counts as one of three Libra correction rounds per PR and restarts the local cycle at round one. Reset only the local counter; preserve the Libra count across heads and ask the user before a fourth batch. Process later Libra results the same way without waiting for future runs to enlarge a batch. Leave thread replies and resolution to Libra.
- Mark the PR ready after completed local-only review or a settled Libra review with no accepted material findings. Verify the published state and latest applicable Libra activity before preparing focus comments.
- Preserve the draft with an explicit caveat when applicability is unresolved or expected Libra results fail, are unavailable, or remain incomplete at the deadline. Preserve the current PR state on correction-limit escalation.

## Pending focus comments and handoff

1. Add a focus comment for each hunk that adds, removes, or changes complex logic: a condition that mixes `and` with `or` across three or more inputs; nested branches that select different results; a loop whose exit or next input depends on earlier iterations; ordering, retry, timeout, or concurrency control; index, offset, time, or money arithmetic beyond a fixed step; a grant or deny decision from two or more inputs; error recovery that rolls back or substitutes data; a migration that rewrites stored data.
2. Treat every other hunk, including config values, docs, and tests, as simple and leave it to local and Libra review. Do not add a comment to describe a hunk or to repeat an accepted finding. Skip hunks the user has already reviewed or removed; do not recreate their focus comments.
3. Write each comment as `Focus:` plus three sentences: what the named symbol decides; the value, count, order of events, or error that makes it hard; one check that names what the user reads and the expected result. Example: "Focus: `splitRange` divides a date range into calendar days. The `2026-03-08` daylight-saving change in `America/New_York` makes one 23-hour day. Check that `splitRange` returns a 23-hour day for `2026-03-08` in `America/New_York`."
4. Apply ASD-STE100 to every sentence: at most 20 words; one statement or one instruction; active voice; present tense; imperative mood for the check; the exact symbol, value, or line from the diff; no noun cluster over three words; one term per concept; no vague words such as boundary, appropriate, properly, ensure, handle, or meaningful. Keep a comment under 40 words. Rewrite a comment that breaks a rule.
5. Read the current head and the user's pending review. When a comment is due, reuse the review or create one if absent, preserve all user comments, and add only missing focus comments with valid diff anchors. If the head changes, reconcile again before writing.
6. Use `gh api` with a JSON payload file; omit `event` when creating the review. Verify `PENDING` state, comment content, and current anchors. Never submit the review. Preserve existing drafts and report any API limitation.
7. Hand off the HTTPS PR link, validation results, review path and Libra outcome, material caveats, and pending focus-comment state: the comment count, or that all hunks are simple. Keep focus guidance in the inline comments and report unavailable comments or checks honestly.

## Tool mechanics

Apply only the subsection for the tool running this session.

### Codex

- Spawn the fresh instances above with `fork_turns="none"`.
- In Plan mode, present the plan inside the required `<proposed_plan>` wrapper.

### Claude Code

- Spawn roles with the `Agent` tool and `subagent_type` set to the role name; never use a fork or the built-in `Plan` and `Explore` subagents for role work.
- In plan mode, write the planner's sections unchanged into the plan file and call `ExitPlanMode` for approval; otherwise present them unchanged and obtain approval before implementing.
- Wait with a background or monitored command instead of foreground `sleep`.

# Core rules

- Read every rule, instruction, and user message word for word. Do not skim.
- Support material technical claims with verified evidence. Use tools when needed and ensure the evidence is current enough for the claim.
- NEVER use hedging language like "most likely", "probably", "generally", or "typically" when the answer is verifiable. Verify it instead.
- When asked about external changes (updates, migrations, upgrades), read the actual changelog or release notes. Do not summarize from general knowledge.
- If something cannot be verified, say "I don't know" rather than guessing.
- Quality and correctness are always more important than speed. Take your time.
- When merging main (or the base branch), regardless of there being merge conflicts or not, always check for semantic conflicts in each of the new commits against the goal of the branch and it's changes.
- When the user corrects a plan, or clarification establishes a concrete change to it, update the actual plan in the same turn. Do not stop at explaining what should change or offering to apply a correction that is already clear and authorized.
- Do not add labels to issues or pull requests unless explicitly asked. Libra draft bypass labels are already authorized in the global workflow.

## PR creation
- Always use "Closes #<issue>" in the PR description, unless the PR does not 
sufficiently close the issue, then use "Part of #<issue>".
- If the PR does not have an issue, then skip issue referencing and do not
create an issue. Simply make sure the PR description describe the intent
properly.
- Always add Ikuni17 as the assignee for PRs and in-progress issues (not when creating new issues)
- Verify repository ownership and fork upstream metadata. Only for repositories owned by `elastic` or forks whose upstream repository is owned by `elastic`, set the following for the linked issue or the direct PR when no issue exists (do NOT set these for PRs that also have a linked issue)
    - Project: Apps DX
    - Project Status: In Progress
    - Iteration: Current Iteration (These are per month iterations, so it will be the current month)
- Set Type: Task for linked issues.
