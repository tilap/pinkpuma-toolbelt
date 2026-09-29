---
name: pr-description
description: >-
  Generate a pull request title and description from the diff between the current
  branch and its base. Use when the user asks to open a PR, write a PR description, or
  prepare a branch for review.
metadata:
  plugin: git-workflow
---

# PR description

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md).

## Steps

1. Determine the base branch (ask if it is not obvious — do not assume `main`) and read
   the full diff: `git log <base>..HEAD --oneline` for the commit list, `git diff
   <base>...HEAD` for the actual change.
2. Draft a title following the same `<type>(<scope>): <description>` shape as a commit
   subject, summarizing the branch as a whole rather than its last commit.
3. Draft the description with, at minimum:
   - **Summary** — what changed and why, in a few bullet points; skip anything the diff
     already makes obvious on its own.
   - **Test plan** — how this was or should be verified (commands run, scenarios
     covered); a checklist of what still needs manual verification if relevant.
4. Show the drafted title and description to the user. **Never run `gh pr create` or
   push the branch without explicit confirmation** — opening a PR is a visible, shared
   action, not something this skill does on its own initiative.

## Notes

- If the repository has a PR template (`.github/PULL_REQUEST_TEMPLATE.md`), fill that
  template's sections instead of inventing a different structure.
