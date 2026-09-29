---
name: handoff
description: >-
  Capture the current session's in-progress, uncommitted work into a handoff document so
  it can be resumed later — in a new session, a different worktree, or by a different
  agent entirely. Use when the user asks to pause, hand off, or save their progress
  before stopping, or before a context-heavy switch of task.
disable-model-invocation: true
metadata:
  plugin: handoff
---

# Handoff

Different from compacting: this is for **leaving**, not for continuing in the same
thread. Write a document another session can pick up cold — complete sentences, no
unexplained shorthand, no assumption the reader watched this session happen.

## What to capture

- **Objective** — what this session was trying to accomplish, in one or two sentences.
- **State of the work** — run `git status --short` and summarize it in prose (what's
  modified, added, untracked); do not paste the raw diff. For anything non-obvious, add
  one line on *why* it's in that state.
- **Decisions made** — choices that would otherwise have to be re-derived (an approach
  tried and rejected, a constraint discovered, a convention picked).
- **Next steps** — the concrete next action, not a vague direction. If there's a known
  blocker, name it.
- **Pointers, not copies** — reference file paths and skills/agents worth re-invoking
  (e.g. "re-run the `project-docs` skill once the API shape settles"). Never duplicate
  file contents into the handoff; the files themselves are the source of truth.

## Where it goes

Write to `.agent/handoffs/<YYYY-MM-DD>-<slug>.md` at the project's root (create the
directory if needed). This location is deliberately not `.claude/` or `.cursor/` — it's
plain project state, readable and resumable regardless of which tool opens it next.

Do not commit the handoff automatically; mention that it was written and let the user
decide whether it belongs in version control for this project.

## Format

```markdown
# Handoff — <one-line objective>

Date: <YYYY-MM-DD>

## State
<prose summary of git status, not a diff dump>

## Decisions
- <decision> — <why>

## Next steps
- <concrete next action>

## Pointers
- <file or skill/agent worth revisiting> — <one line on why>
```
