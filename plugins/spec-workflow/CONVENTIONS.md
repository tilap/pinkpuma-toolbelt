# Spec workflow conventions

Shared by every skill in this plugin. Read this once; each `SKILL.md` links back here
instead of repeating it.

## The chain

```
to-brief  →  to-spec  →  to-tickets  →  (implementation, one ticket per fresh session)
```

| Skill        | Decides? | Output                                                     | Lives in                       |
| ------------ | -------- | ---------------------------------------------------------- | ------------------------------ |
| `to-brief`   | Yes      | Business brief, including drafted ADRs and doc updates     | `.agent/specs/<slug>/brief.md` |
| `to-spec`    | No       | Technical spec, published as the parent issue              | GitHub issue + local copy      |
| `to-tickets` | No       | Vertical-slice tickets with blocking edges and doc updates | GitHub sub-issues of the spec  |

Only `to-brief` makes decisions. `to-spec` and `to-tickets` record and slice what was
already decided: if they need a decision nobody made, they stop and send the user back
to `to-brief` rather than inventing one.

Skip steps that buy nothing:

- The need fits one working session once briefed → skip `to-spec`; `to-tickets` turns
  the brief into a single ticket.
- Several sessions, but the brief already settles the technical shape → `to-spec` is
  still worth it: it is the one durable place the reasoning behind the tickets lives.

The chain ends when the tickets are on GitHub. Implementation and review happen
elsewhere, later, from those tickets.

## The repository stays untouched

These skills prepare work; they never do it. None of them edits, creates or deletes a
file the repository tracks — not code, not `docs/`, not `.gitignore`. The only local
writes are:

- the working folder `.agent/specs/<slug>/`;
- one line, `.agent/specs/`, in `.git/info/exclude` (git's local, untracked ignore file),
  added if missing, so the working folder never shows up in `git status`.

Everything else ends up in GitHub issues. Implementation, including writing the
documentation the need calls for, happens later in its own sessions.

## Working files

Each need gets one folder, `.agent/specs/<slug>/`, at the project root (`<slug>` is
kebab-case, 2–5 words, named after the need, not the solution):

- `brief.md` — written by `to-brief`.
- `spec.md` — local copy of the published spec, with the issue number on its first line.

They exist so a later step can run in a fresh session and read the whole thing back,
instead of depending on a conversation that was cleared or an issue body too long to
fetch cleanly. They stay with the local clone — deleting the project deletes them.

Before the first write, make sure `.git/info/exclude` contains `.agent/specs/`; append
the line if it does not. If the project is not a git repository, skip this.

Once the tickets are published, GitHub is the source of truth and the folder is
disposable: `to-tickets` offers to delete it.

## Reading the project documentation

Before drafting anything, read what the project already settled. The expected layout:

| Path                     | Read it for                                                                  |
| ------------------------ | ---------------------------------------------------------------------------- |
| `docs/context/`          | Product purpose, actors, business rules — and the **vocabulary** to write in |
| `docs/architecture/`     | Modules, boundaries, data model, the seams where behaviour can be tested     |
| `docs/decisions/`        | `ADR-NNNN-kebab-title.md` records and their `README.md` index — constraints  |
| `docs/operations/`       | Deployment, runbooks, environments — non-functional constraints              |
| `README.md`, `AGENTS.md` | Stack and commands                                                           |

A project documented differently wins: map the same intentions onto what is there
(`docs/PRODUCT.md`, `docs/ARCHITECTURE.md`, `docs/DECISIONS.md`, a single
`DOCUMENTATION.md`, …). If a document is missing, proceed silently — never block on it
and never create documentation up front.

Read only what touches the need: every ADR title in the index, then the full text of the
ADRs and documents in the area being changed. Treat a document with `status: stale` in
its frontmatter as a hint, not a fact — check it against the code.

### Vocabulary

There is no glossary file. The project's vocabulary is the words `docs/context/` and the
code already use for domain concepts. Write every artifact in those words. When the user
or a draft uses a different word for an existing concept, say so and use the project's
word. When a concept has no name yet, the brief names it once, in its Vocabulary section.

### ADR conflicts

If anything you produce contradicts an accepted ADR, say so explicitly instead of
silently overriding it:

> _Contradicts ADR-0007 (event-sourced orders) — reopening it because …_

## Documentation updates are work, not writes

A need often calls for documentation changes — an ADR, a new business rule in
`docs/context/`. These skills **draft** them; they never write them into `docs/`. The
draft travels down the chain and lands in a ticket, and whoever implements that ticket
writes it in the same pull request as the code it describes:

1. `to-brief` drafts each update the user agreed to, in full, in the brief's
   Documentation updates section.
2. `to-spec` carries the drafts over unchanged.
3. `to-tickets` attaches each draft to the ticket that makes it true, with an acceptance
   criterion that it is written — so a later review can check it.

### ADRs

Propose an ADR only when all three hold:

1. **Hard to reverse** — changing your mind later has a real cost.
2. **Surprising without context** — a future reader would wonder why it was done this way.
3. **A real trade-off** — there were genuine alternatives, and one was picked for reasons.

Most needs produce zero ADRs; that is normal. When one qualifies, draft it with the
instructions the implementer will need:

- **Number**: leave `ADR-NNNN` as a placeholder. The implementer takes the next free
  number at writing time — other work may claim numbers in between.
- **File**: `docs/decisions/ADR-NNNN-kebab-title.md`.
- **Shape**: match the project, in this order — `docs/decisions/ADR-TEMPLATE.md` if it
  exists; otherwise the most recent existing ADR (same frontmatter fields, same
  sections); only in a project with no ADR yet, the shape below.
- **Index**: the implementer adds an entry to `docs/decisions/README.md`, matching its
  existing format.
- **Superseding**: name the ADR it replaces; the implementer sets that ADR's
  `status: superseded` and `superseded_by`, and updates its index entry.

```markdown
---
title: <Short decision title>
summary: <One line: what was decided>
status: accepted            # proposed | accepted | rejected | superseded | deprecated
date: <YYYY-MM-DD>
---

# ADR-NNNN: <Short decision title>

## Context

<What forced a decision, in 1–3 sentences.>

## Decision

<What was decided and why, in 1–3 sentences.>

## Consequences

<Only if non-obvious: what this makes easier, harder, or forbidden.>

## Alternatives considered

<Only if a rejected option is likely to be suggested again: what it was and why not.>
```

Keep it short: the value is recording *that* a decision was made and *why*.

### Context documents

When the brief settles a **lasting business rule or term** that `docs/context/`
contradicts or lacks, draft the exact edit: which document, which section, the text to
add or replace.

Only lasting truths become documentation updates. The feature itself — plans, scope,
acceptance criteria — belongs in the brief and the tracker, never in `docs/`, which
describes the present and decided history.

If `docs/INDEX.md` exists and is generated, the ticket carrying a documentation update
also says it must be regenerated rather than hand-edited.

## GitHub

All tracker operations go through the `gh` CLI, run inside the clone (the repository is
inferred from the git remote). Check `gh auth status` before the first write.

| Operation                  | Command                                                                     |
| -------------------------- | --------------------------------------------------------------------------- |
| Read an issue              | `gh issue view <n> --json number,title,body,labels,comments`                |
| Search for overlap         | `gh issue list --state all --search "<keywords>" --json number,title,state` |
| List labels                | `gh label list --json name`                                                 |
| Create (body from a file)  | `gh issue create --title "…" --body-file <file>`                            |
| Create as a sub-issue      | add `--parent <n>`                                                          |
| Create with blocking edges | add `--blocked-by <n>,<n>`                                                  |

`--parent` and `--blocked-by` need a recent `gh`. If they are rejected, fall back to
`Part of #<n>` as the body's first line and a `## Blocked by` section listing `#<n>`
references.

### Labels

- Parent spec issue: `spec`.
- Tickets: `ready-for-agent`.

Apply a label only if it already exists in the repository. If it does not, ask once
whether to create it; never create labels silently. The spec never gets
`ready-for-agent`: an agent that polls for that label would try to build the whole spec
in one run instead of its tickets.

### Publishing rules

- **Nothing is published without the user's explicit approval** of the exact content.
  Creating issues is visible to everyone with access to the repository.
- Search for overlapping open issues first, and show any hit to the user before creating.
- Never close, retitle or rewrite an existing issue the user did not ask you to change.

## Writing rules for everything published

- **Language**: the language of the project's existing documentation; if there is none,
  the language of the conversation.
- **Never invent.** Every statement traces back to the brief, the conversation, the
  documentation or the code. A gap goes into an "Open questions" list, visibly — never
  smoothed over with a plausible guess. Precise answers stay precise: a number, an
  ordering guarantee or an explicit "must not" is copied as-is, not paraphrased.
- **Durable over precise.** No file paths, no line numbers: they go stale before the
  ticket is picked up. Name modules, types, interfaces and behaviours instead. Two
  exceptions: paths under `docs/` in a documentation update (they say where to write),
  and a short snippet from a prototype that encodes a decision better than prose (a state
  machine, a schema, a type shape) — trim it to the decision and say where it came from.
- **Behaviour, not procedure.** Say what the system must do, not which function to edit.
