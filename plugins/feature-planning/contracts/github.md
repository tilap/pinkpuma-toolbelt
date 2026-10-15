# Contract: GitHub features and tickets

What the automated development system reads when features and tickets are on GitHub, and
what follows from each choice. Distilled from the system's own contract document (marked
there as temporary): if the system changes, this file is what to update.

The system builds software from issues, unattended. It does **not** split work, write
issues or judge whether a plan is good: everything it does is decided by what the issues
say. A bad ticket produces bad work, silently.

## Shape

```
Feature  (issue, label feature-ready)
 ├─ Ticket A  (sub-issue, label task-ready)
 ├─ Ticket B  (sub-issue, label task-ready, blocked by A)
 └─ Ticket C  (sub-issue, label task-ready, blocked by B)
```

- Feature and tickets are in **one repository**. Cross-repository sub-issues and blockers
  are not handled.
- Tickets are the **native sub-issues** of the feature — not a task list, not issues that
  merely mention it. Only direct sub-issues count; at most 1000 are read.
- One feature at a time per project. The system opens branch `feature/<id>`, does the
  tickets one by one (each its own pull request, checked, then merged into that branch),
  then opens one pull request to the final branch, which a human merges unless the feature
  carries `merge-when-ready`.

## Labels

| Label              | On               | Meaning                                                                   | Set by                                                             |
| ------------------ | ---------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `feature-ready`    | Feature          | Content final, tickets may be read (**only labelled issues are listed**)  | the creator, last                                                  |
| `task-ready`       | each Ticket      | The ticket's specification is final                                       | the creator, on all tickets together, after the list was validated |
| `merge-when-ready` | Feature          | The final pull request may be merged without a human                      | creator or human                                                   |
| `to-resume`        | Feature / Ticket | Resume after an escalation                                                | a human                                                            |
| `project:<name>`   | Feature          | The project. A ticket takes its feature's. Missing: a configured default  | the creator                                                        |
| `priority:<n>`     | Feature / Ticket | `0`–`100`, or `low` 25, `normal` 50, `high` 75, `urgent` 100. Missing: 50 | the creator                                                        |
| `mason:*`          | any              | State labels the system writes. **Never set or remove them**              | the system                                                         |

The two labels may share one name (for example `ready-for-agent` for both): a feature is a
top-level issue and a ticket is a sub-issue, so the same label is read on each side without
confusion. Everything below applies to either choice.

**Label names.** The names above are this plugin's defaults, kept in this one file. The
system can be configured with other names, but the plugin cannot see that configuration, so:
a name the user gives in the request wins; before applying any label, list the repository's
labels (`gh label list`) and compare. A label that does not exist is reported, never created
silently. In particular, `priority:*` labels are not created by the system's setup, so they
may be missing on a fresh repository.

## The feature issue

- **Title**: a real title. It is the final pull request's title (`(#N)` appended, trailing
  full stop dropped) and part of what the system compares to detect changes.
- **Body**: read word for word, **no structure extracted** (no headings, checklist or
  criteria). It is the statement of the need and the final pull request's text, written for
  the reviewer who will merge. It is *not* what the agent codes from — tickets are.
- **Refused as `invalid`** (one comment, nothing starts): no title and no body; more than
  one `project:` label, or none and no default; more than one `priority:` or a value off
  the scale; it is a pull request.
- **Order**: highest priority, then oldest, among unblocked features.
- **Dependencies between features**: GitHub's native *Blocked by*. A feature opens only when
  every blocker is `done` in the system's ledger — closing a blocker by hand unblocks nothing.

## The ticket issue

A ticket is one unit of work: one agent run (with a few repair attempts), then gates, then a
pull request. It is never split further. **The body is the agent's task, verbatim.** The
title is the fallback when the body is empty and the title of the ticket's pull request.

The agent has the repository and nothing else: it does not see the feature, the other
tickets, or any document. Therefore each ticket body must:

- be **self-contained**: what to change and where it matters, in the repository's own words;
- state **observable outcomes** — what a person or a test can check; the reviewing agent and
  the checks judge outcomes, not intentions;
- leave the repository's own checks **green on its own**: the ticket is merged into the
  feature branch before the next one starts, so none may rely on a later one to compile or
  pass;
- be small enough for one run: one behaviour, one slice.

There is no acceptance-criteria field: say it in the body. Ticked boxes are fine.

- **Dependencies**: native *Blocked by* on the sub-issue, no cycle. A ticket starts when all
  its blockers are `done`; among those that can start: highest priority, then oldest. If
  nothing can start while tickets remain (cycle, cancelled blocker), the feature escalates.
- **Priority**: `priority:` label, default 50; it only orders ready, unblocked tickets.
- **A ticket the system cannot read is dropped silently** and the feature proceeds without
  it: two `priority:` labels, no title and no body, a bad value. Validate before labelling.

## When work starts

A feature opens when **all** hold: it carries `feature-ready`; every feature blocking it is
`done`; no other feature is open on the project; it has at least one ticket and **every**
ticket carries `task-ready`.

- Tickets are read **once, when the feature opens**. A ticket added later is ignored.
- Edits after work started are not applied; to change running work, close it and write a
  new issue. Comments are never read as instructions.

**Order of creation** — so nothing opens on a half-written list:

1. Create the feature **without** `feature-ready`.
2. Create all tickets as sub-issues, with blockers and priorities.
3. A human validates the list.
4. Add `task-ready` to every ticket, together.
5. Add `feature-ready` to the feature, **last**.

Steps 4 and 5 start unattended development: the moment they are done, the system may begin.

## What the system writes back

Comments ending with a machine marker (`<!-- ticket-event … -->`), `mason:*` labels,
branches `feature/<id>` and `issue/<id>`, pull requests and squash merges. A ticket issue is
**not** closed when its pull request merges into the feature branch; the feature issue is
closed by the final merge. Anything that watches these issues must ignore those comments and
labels. Closing an issue cancels it; closing a feature cancels its unfinished tickets.

## Known limits

- Only `done` in the system's ledger unblocks a dependency.
- A feature none of whose tickets landed escalates; one that lost some lands without them.
- The flow has run for real on one feature of one ticket; chains of blockers, two features in
  a row and `merge-when-ready` are covered only by tests against simulated GitHub.

## Checklist before publishing

Feature:

- [ ] a real title; a body that says what and why, for a reviewer
- [ ] `project:<name>` (or the default applies); at most one `priority:`
- [ ] *Blocked by* set if another feature must land first
- [ ] `feature-ready` added **last**

Each ticket:

- [ ] a native sub-issue of the feature, in the same repository
- [ ] a title, and a self-contained body with observable outcomes
- [ ] leaves the checks green when merged alone
- [ ] *Blocked by* set to the tickets it needs, no cycle
- [ ] at most one `priority:` label
- [ ] `task-ready` added together with its siblings
