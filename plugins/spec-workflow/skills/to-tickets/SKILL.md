---
name: to-tickets
description: >-
  Break a spec, brief, or the current conversation into vertical-slice GitHub issues —
  each demoable on its own and sized for one fresh session — published as sub-issues of
  the spec with native blocking links. Use after to-spec, or on a plan already settled,
  when the work spans several sessions.
disable-model-invocation: true
metadata:
  plugin: spec-workflow
---

# To tickets

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md) — read it first (chain, working
files, GitHub, writing rules). The repository stays untouched: this skill publishes
issues and nothing else. It is the last step of the chain — implementation and review
happen elsewhere, later, from the tickets.

One job: cut decided work into tickets a fresh session can pick up and finish alone,
each declaring the tickets that **block** it. Like `to-spec`, this skill decides
nothing: if slicing needs a decision nobody made, stop and send the user back to
`to-brief`.

## Step 1 — Gather the source

In order of preference: the local copy `.agent/specs/<slug>/spec.md` (its first line
names the spec issue), a spec issue passed as argument (read body and comments), the
brief, the current conversation.

## Step 2 — Explore the code

Read the code in the area and the ADRs that constrain it, so tickets use the project's
vocabulary and module names. Look for **prefactoring** — changes that make the feature
easy to add without changing behaviour ("make the change easy, then make the easy
change"). Prefactoring goes first in the order, as its own tickets.

## Step 3 — Draft vertical slices

Each ticket is a **tracer bullet**:

- It cuts a narrow but **complete** path through every layer the feature touches (data,
  logic, interface, tests) — never one layer on its own.
- It is **demoable** or verifiable on its own once merged: you can name what to show.
- It fits in **one fresh session** that has never seen this conversation.

Size against two failures, both common:

- **Horizontal slicing** — "all the schema", then "all the API", then "all the UI".
  Test every ticket with: *what can I demo when this one is done?* No answer means it
  is a layer, not a slice — merge or re-cut.
- **Over-decomposition** — a dozen tickets for a change one session could finish. If
  the whole thing fits one session, say so and propose a single ticket (or none). A
  ticket too small to demo anything new should be merged into a neighbour.

Give each ticket its **blocking edges**: the tickets that must be done before it can
start. Keep only edges that genuinely gate the work; a ticket with no blockers can start
immediately, and the more of those, the more can run in parallel.

**Documentation updates.** Attach every `DU` draft from the spec (or brief) to the
ticket whose behaviour makes it true — usually the first ticket the decision shapes —
never to a documentation-only ticket of its own. Copy the draft in full into that
ticket, with a criterion that it is written. Every `DU` lands in exactly one ticket.

**Wide-refactor exception.** One mechanical change whose blast radius covers the
codebase (rename a column, retype a shared symbol) cannot land as a green vertical
slice. Sequence it **expand → migrate → contract** instead: add the new form beside the
old; migrate callers in batches sized by blast radius (one ticket each, blocked by the
expand); delete the old form in a ticket blocked by every batch.

## Step 4 — Make every acceptance criterion able to fail

For each criterion, name the observation that would show it false, and check that it:

- **fails at the starting commit** — a criterion already true grades nothing;
- **is satisfiable inside this ticket** — not only by work another ticket owns;
- **derives from the spec** — cite the rule or scenario (`R2`, `S4`) it comes from.

Rewrite or drop any criterion that misses one of the three.

## Step 5 — Review with the user

Present the breakdown as a numbered list, in dependency order:

```
1. <Title> — blocked by: none — demo: <what can be shown when done>
2. <Title> — blocked by: 1 — demo: …
```

Then ask: Is the granularity right (too coarse / too fine)? Is every blocking edge real?
Should any ticket be merged or split? Iterate until the user approves the list. Nothing
is published before that.

## Step 6 — Publish

Publish in dependency order, blockers first, so each ticket can reference real issue
numbers:

- `gh issue create --parent <spec> --blocked-by <n>,<n> --label ready-for-agent
  --body-file <file>` (labels and fallbacks: see Conventions).
- With no spec issue (sliced from a brief or the conversation), omit `--parent`.
- Do not close or modify the spec issue.

Then report, and stop:

- each ticket's number, title and blockers;
- the **frontier**: the tickets with no open blocker, which can start now;
- which ticket carries each documentation update.

Finally, offer to delete `.agent/specs/<slug>/`: GitHub now holds everything it
contained. Delete it only on a yes.

If the repository has no GitHub remote or `gh` is unavailable, stop before publishing
and say so; nothing is written anywhere else instead.

## Ticket template

```markdown
## What to build

<The end-to-end behaviour this ticket makes work, from the user's perspective — not a
layer-by-layer task list.>

## Demo

<What to run or show to see it working once merged.>

## Acceptance criteria

- [ ] <Criterion> (R1)
- [ ] <Criterion> (S3)
- [ ] <DU1 written as drafted below, with its index entry> (DU1)

## Documentation updates

<Only when this ticket carries a DU: the full draft from the spec, to write in the same
pull request as the code. Omit otherwise.>

## Out of scope

- <What this ticket must not touch — usually the next ticket's behaviour>

## Blocked by

<Fallback only, when native blocking links are unavailable: #<n>, #<n>. Omit otherwise.>
```

Add `Part of #<spec>` as the first line only when `--parent` was unavailable.
