---
name: to-tickets
description: >-
  Break a feature spec (or the current conversation) into vertical-slice, self-contained
  tickets that each leave the project green on their own, publish them to the ticket
  tracker with their blocking links, and — on separate explicit approval — release them to
  the automated development system. Use after to-spec, or on a plan already settled.
disable-model-invocation: true
metadata:
  plugin: feature-planning
---

# To tickets

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md) — read it first (the three
places, nothing written to disk, publishing and writing rules). **Then read the contract
for the ticket tracker** in [../../contracts/](../../contracts/): the tickets are consumed
by an automated system that drops what it cannot read, so its rules are requirements here.
No contract file for the tracker in use → say so and ask the user for its rules before
publishing; drafting is still fine.

One job: cut decided work into tickets a coding agent can pick up and finish alone, each
declaring the tickets that **block** it. Like `to-spec`, this skill decides nothing: if
slicing needs a decision nobody made, stop and send the user back to `grill`.

## Step 1 — Gather the source

In order of preference: the spec from this conversation; a feature the user points at (read
its body and comments from the board); the **Settled so far** of `grill`; the conversation.
Confirm the tracker, the board and the code location (see Conventions).

## Step 2 — Explore the code

Read the code in the area (read-only) and the ADRs that constrain it, so tickets use the
project's words and point at the right places. Look for **prefactoring** — changes that make
the feature easy to add without changing behaviour. Prefactoring goes first, as its own
tickets.

## Step 3 — Draft vertical slices

Each ticket is a **tracer bullet**:

- It cuts a narrow but **complete** path through every layer the feature touches (data,
  logic, interface, tests) — never one layer on its own.
- It is **demoable or checkable alone** once merged: you can name what to show.
- It leaves the repository's own checks **green on its own**: tickets merge one after the
  other, so none may rely on a later one to compile or pass.
- It fits **one agent run** with no memory of this conversation or of the spec.

Test every ticket with *what can I check when this one is done?* No answer means it is a
layer, not a slice — merge or re-cut. Avoid over-decomposition too: if the whole need fits
one run, propose a single ticket. A ticket too small to check anything new belongs in a
neighbour.

Give each ticket its **blocking edges**: the tickets that must be done first. Keep only
edges that genuinely gate the work; no cycles. More tickets with no blockers means more
freedom of order.

**Order versus priority.** A build order that must hold (schema before the screens that
read it, an API before its client) is a **blocking edge**, never a priority: a priority only
orders tickets that are already ready and unblocked, so it cannot enforce anything. Set a
`priority` only when several tickets can start at once and one should go first (a risky
spike before routine work, say), and set it on those tickets alone. Leave every other ticket
without a priority label: the default applies. When in doubt between the two, use a block.

**Documentation updates.** Attach each draft from the spec to the ticket whose behaviour
makes it true — never to a documentation-only ticket — in full, with a criterion that it is
written. Every draft lands in exactly one ticket.

**Wide-refactor exception.** One mechanical change whose blast radius covers the codebase
cannot land as a green vertical slice. Sequence it **expand → migrate → contract**: add the
new form beside the old; migrate callers in batches (one ticket each, blocked by the
expand); delete the old form in a ticket blocked by every batch.

## Step 4 — Write each body as the agent's whole world

The agent sees the repository and the ticket body, nothing else: not the feature, not the
spec, not the other tickets. So each body:

- restates, in the repository's own words, the part of the spec this ticket needs — copy
  the relevant rules and scenarios (`R2`, `S4`) in, do not link to them;
- states **observable outcomes** a person or a test can check, not intentions;
- may name the few paths that matter (where to change, which test to extend), without line
  numbers or code (see Conventions);
- says what the ticket must **not** touch, usually the next ticket's behaviour.

Then make every acceptance criterion able to fail: name the observation that would show it
false, and check that it fails at the starting commit, is satisfiable inside this ticket
alone, and derives from the spec. Rewrite or drop any that misses one.

## Step 5 — Review with the user

Present the breakdown as a numbered list in dependency order:

```
1. <Title> — blocked by: none — priority: <value, or "none"> — checkable: <what shows it works>
2. <Title> — blocked by: 1 — priority: none — checkable: …
```

Then ask: is the granularity right? Is every blocking edge real? Does every required
build order appear as a block rather than a priority? Should any ticket be merged or split? Iterate until the user approves the list. Nothing is published before that.

## Step 6 — Validate before publishing

Run the contract's checklist over every ticket and over the feature, and report the result
as a short table (ticket × check). Fix what fails. In particular, check what the system
would drop silently: missing title or body, more than one priority label or a value off the
contract's scale, a cycle, a ticket that relies on a later one. Also check that no required
order rests on priorities alone.

## Step 7 — Publish the tickets

With the user's explicit approval of the exact content, create the tickets in dependency
order, blockers first, so each can reference real numbers:

- **Same system as the feature, native links available** (GitHub): create each as a
  sub-issue of the feature with its blockers — `gh issue create --parent <feature>
  --blocked-by <n>,<n> --title … --body-file -` — plus a `priority:` label only on the
  tickets whose draft carries one. **Do not add `ready-for-agent`** yet.
- **Feature on another system**: create the tickets in the tracker and, as the last line of
  each body, the feature's URL as a plain reference. Say that the downstream contract for
  this combination is not known; do not release (Step 8).
- Do not close or modify the feature or any existing item.

Report each ticket's number, title and blockers, the tickets with no blocker (the frontier),
and which ticket carries each documentation update. Then stop and ask for the validation of
the list as it now stands on the tracker.

## Step 8 — Release to the development system (separate approval)

Only when the contract says labels start development, and only after the user has looked at
the published tickets and says go. Explain what will happen first: **from the moment the
labels are in place, development may start unattended.** On approval, in this order:

1. Add the "tickets final" label (GitHub: `ready-for-agent`) to **every** ticket, together.
2. Add the "feature final" label (GitHub: `ready-for-agent`) to the feature, **last**.

Never set or remove labels the system manages (GitHub: `mason:*`). A ticket added after the
release is ignored by the system; if the list must change, tell the user rather than adding.

## Ticket template

```markdown
## Context

<The part of the need this ticket serves, in the repository's own words — enough for an
agent that has seen nothing else.>

## What to build

<The end-to-end behaviour that must work, not a layer-by-layer task list.>

## Where

<Optional. The few paths that matter: where to change, which test to extend.>

## Done when

- [ ] <Observable outcome a person or a test can check> (R1)
- [ ] <Observable outcome> (S3)
- [ ] <Documentation update written as drafted below, with its index entry> (DU1)
- [ ] The project's own checks pass.

## Documentation update

<Only when this ticket carries one: the full draft, to write in the same pull request as
the code. Omit otherwise.>

## Out of scope

- <What this ticket must not touch>
```
