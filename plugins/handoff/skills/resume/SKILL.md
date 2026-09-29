---
name: resume
description: >-
  List available handoff documents for the current project and load the most recent (or
  a specified) one into context, falling back to reconstructing state from git when no
  handoff was written. Use when starting a session on a project that has prior handoffs
  or uncommitted work, or when the user asks to resume previous work.
metadata:
  plugin: handoff
---

# Resume

## When a handoff exists

1. Look for `.agent/handoffs/` at the project root.
2. List the handoff files found, newest first (the `<YYYY-MM-DD>-<slug>.md` filename
   sorts correctly). If there is more than one, show the list and ask which one, unless
   the user already named a date or topic.
3. Read the chosen handoff and treat it as prior context, not as instructions to execute
   blindly — the world may have moved since it was written. Cross-check its "State"
   section against a fresh `git status --short` before acting on anything it says is
   still in progress; if they disagree, say so and ask before proceeding.
4. Start from its "Next steps" section.

## When none exists, but there's uncommitted work

Don't just say "nothing to resume" if `git status --short` shows changes — reconstruct
instead of giving up:

1. Run `git status --short` and `git diff` to see what's modified, added, or untracked.
2. Check recent commit messages (`git log --oneline -10`) for the thread this work
   continues.
3. Summarize a best-effort picture of what looks `DONE` (committed and consistent),
   `PARTIALLY DONE` (modified but incomplete — a half-written function, a TODO, a failing
   test), and `NOT DONE` (clearly intended but not started, if inferable from comments or
   an issue reference).
4. Say explicitly that this is a reconstruction, not a record the user wrote — lower
   confidence than an actual handoff, and worth a quick confirmation before acting on it.

Only when both `.agent/handoffs/` is empty or missing *and* the working tree is clean is
there genuinely nothing to resume.
