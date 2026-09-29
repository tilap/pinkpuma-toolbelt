---
name: branch-name
description: >-
  Derive a Conventional-Commits-style branch name from a short description or a ticket
  reference. Use when the user asks for a branch name, wants to start a new branch, or
  says what they are about to work on before creating one.
metadata:
  plugin: git-workflow
---

# Branch name

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md).

## Steps

1. Identify the type of work (`feat`, `fix`, `chore`, …) from what the user described —
   ask only if it is genuinely ambiguous, not by default.
2. Reduce the description to 3–6 kebab-case words that would let a reader understand the
   change from the branch name alone. Drop filler words (`the`, `add-a-new`, `for`) and
   ticket boilerplate; keep the ticket ID only if the team's convention requires it
   (check `AGENTS.md`/`CLAUDE.md` first — a project's own convention wins).
3. Propose the branch name: `<type>/<kebab-description>`.
4. Only create and switch to the branch if the user asks for that explicitly — proposing
   a name and creating/switching branches are different levels of commitment, and
   switching away from uncommitted work on the current branch needs a `git status` check
   first.
