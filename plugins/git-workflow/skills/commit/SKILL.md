---
name: commit
description: >-
  Draft a Conventional Commits message from the staged/unstaged diff, then create the
  commit. Use when the user asks to commit changes, write a commit message, or asks what
  the commit message should be.
disable-model-invocation: true
metadata:
  plugin: git-workflow
---

# Commit

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md).

## Steps

1. Read the actual change, not just file names: run `git status --short` and
   `git diff` (and `git diff --staged` if something is already staged). If nothing is
   staged, stage only the files that belong to this change — never `git add -A` or
   `git add .` blindly, and call out anything that looks unrelated before including it.
2. Classify the change against the commit types in `CONVENTIONS.md`. If the diff mixes
   unrelated concerns (e.g. a feature plus an unrelated formatting pass), say so and
   propose splitting into separate commits rather than picking one type that half-fits.
3. Draft the subject line: `<type>(<scope>): <description>`, imperative mood, under
   ~72 characters. Add a body only when the *why* is not obvious from the diff itself —
   a hidden constraint, a workaround, a decision that would otherwise need re-deriving.
   Never restate what the diff already shows.
4. Show the drafted message to the user before committing, unless they explicitly asked
   to skip confirmation for this session.
5. Commit with the approved message. Never pass `--no-verify` or otherwise skip hooks
   unless the user explicitly asked for that.

## Notes

- If a repository's own `CONTRIBUTING.md` or `AGENTS.md`/`CLAUDE.md` states a different
  commit convention, follow that instead and mention the divergence once — never impose
  this plugin's format over a project's own documented one.
- This skill only drafts and commits locally. Pushing, opening a PR, or amending a commit
  that is already on a shared branch are separate, more sensitive actions — do not chain
  them in automatically.
