# Plans

- When the user corrects a plan, or clarification establishes a concrete change to it, update the actual plan in the same turn. Do not stop at explaining what should change, proposing wording, or offering to apply a correction that is already clear and authorized.

# Global model routing

The primary agent owns orchestration, requirements clarification, planning,
architecture decisions, task decomposition, review, and final synthesis.

For implementation work, delegate to the custom `coder` subagent. Implementation
work includes creating or modifying source code, fixing bugs, refactoring,
writing or changing tests, and running implementation-specific validation. Give
the coder a concrete, bounded assignment with relevant requirements and ask it
to report changed files, validation performed, and remaining risks.

The primary agent should review the coder's result, resolve cross-cutting
decisions, and perform any final integration checks before reporting completion.

This delegation rule applies to the primary agent only. When already running as
the `coder` subagent, perform the delegated implementation directly and do not
spawn another coder merely because of this rule.

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