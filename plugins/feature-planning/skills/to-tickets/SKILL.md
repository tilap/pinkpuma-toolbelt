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

**Specify the tests, so they can be written first.** The tests of a ticket are written
*before* its code, by an agent that sees only this body and is not the one that writes the
code; the code must then make them pass without editing them. So the ticket defines them —
as cases, never as test code — and the reviewer can match criterion to test (gate B in
[REVIEW.md](../../REVIEW.md)). In a **Tests** section, give:

- one or more **cases** per criterion, as Given / When / Then: the nominal path, the edges
  (empty, limit, duplicate), and the refusals the spec requires. Each case names the rule
  or scenario it covers (`R2`, `S4`);
- **one data set per case, and a Given that makes the Then observable.** A Then with a
  number in it (a page of 50, a cap of 200, 3 rows kept by a filter) names in its Given the
  data that produces that number, past any bound: "Given 210 notes on one day, when
  listing with limit 500, then 200" — never "Given 60 notes … then at most 200", which no
  seed of 60 can show. A criterion that needs two sets of data (60 rows for the page, more
  than 200 for the cap) is two cases. The agent that writes the tests seeds exactly what
  the Given says; a Given it has to complete by guessing is where the tests go wrong;
- the **seam and the interface under test**: the public function, endpoint or command, with
  its name, signature, inputs and outputs. Tests are written against something that does not
  exist yet; without the interface they would fail on a missing import instead of on the
  missing behaviour, and prove nothing;
- **where** they go: the test file to add or extend and the framework already in use;
- what a case does at the starting commit: it fails because the behaviour is absent.

A ticket that cannot be tested first — styling, configuration, documentation, a
behaviour-preserving prefactoring — says `Tests: none — <reason>` and names the existing
check that guards it. Never leave the section out: "no tests" is a choice that shows. Add
the **cross-cutting** outcomes the ticket really owes —
a reversible migration, a permission enforced, a limit applied, a dependency justified —
and only those it owes. Tell the agent what to do when stuck: if the ticket is unclear or
contradicts the code, stop and say so rather than choose.

## Step 5 — Review with the user

Present the breakdown as a numbered list in dependency order:

```
1. <Title> — blocked by: none — priority: <value, or "none"> — checkable: <what shows it works> — tests: <n cases, or "none — reason">
2. <Title> — blocked by: 1 — priority: none — checkable: … — tests: …
```

Then ask: is the granularity right? Is every blocking edge real? Does every required
build order appear as a block rather than a priority? Should any ticket be merged or split? Iterate until the user approves the list. Nothing is published before that.

## Step 6 — Validate before publishing

Run the contract's checklist over every ticket and over the feature, and report the result
as a short table (ticket × check). Fix what fails. In particular, check what the system
would drop silently: missing title or body, more than one priority label or a value off the
contract's scale, a cycle, a ticket that relies on a later one. Also check that no required
order rests on priorities alone, and that every ticket has a Tests section (cases with
Given / When / Then and the interface under test named, each numeric Then produced by the
data its own Given seeds) or an explicit `Tests: none` with its reason and the check that
guards it.

Then build the **review sheet**, the input of the plan review and of the final review:

- A **matrix**, one row per rule `R*`, scenario `S*` and documentation update `DU*` of the
  spec: the ticket(s) that carry it and the case of the ticket's Tests section (seam and interface) that proves it. A row with
  no ticket is a gap; a ticket criterion with no row is an orphan. Fix both before going on.
- For a feature without a spec (a single ticket), one row per "Done when" line.

Run **gate A** of [REVIEW.md](../../REVIEW.md) over the drafts, answering every item
`pass` / `fail` / `n/a` with its evidence, and show it with the matrix. The user is the
reviewer of this gate: they confirm or overrule each `fail`. Nothing moves on while an item
is `fail`.

## Step 7 — Publish the tickets

With the user's explicit approval of the exact content, create the tickets in dependency
order, blockers first, so each can reference real numbers:

- **Same system as the feature, native links available** (GitHub): create each as a
  sub-issue of the feature with its blockers — `gh issue create --parent <feature>
  --blocked-by <n>,<n> --title … --body-file -` — plus a `priority:` label only on the
  tickets whose draft carries one. **Do not add the ticket-ready label** yet.
- **Feature on another system**: create the tickets in the tracker and, as the last line of
  each body, the feature's URL as a plain reference. Say that the downstream contract for
  this combination is not known; do not release (Step 8).
- Do not close or edit the feature or any existing item (the review sheet below is a new
  comment, not an edit).

- With the user's approval of its exact text, post the **review sheet** (the matrix, with
  the published ticket numbers, then the checks of gates B and C that the matrix feeds) as
  **one comment on the feature**. The final reviewer reads it there. It is a comment: the
  system never reads comments as instructions, so it changes nothing about the work.

Report each ticket's number, title and blockers, the tickets with no blocker (the frontier),
and which ticket carries each documentation update. Then stop and ask for the validation of
the list as it now stands on the tracker.

## Step 8 — Release to the development system (separate approval)

Only when the contract says labels start development, and only after the user has looked at
the published tickets and says go. Explain what will happen first: **from the moment the
labels are in place, development may start unattended.** On approval, in this order:

1. Add the "tickets final" label (GitHub default: `task-ready`) to **every** ticket, together.
2. Add the "feature final" label (GitHub default: `feature-ready`) to the feature, **last**.

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
- [ ] <Cross-cutting outcome this ticket owes, if any>
- [ ] <Documentation update written as drafted below, with its index entry> (DU1)
- [ ] Each outcome above is covered by a case of Tests below, written before the code and
      left unchanged by it.
- [ ] The project's own checks pass, none skipped or weakened.

## Tests

Written first, by an agent that sees only this ticket; the code then makes them pass without
editing them.

- **Interface under test**: <public function, endpoint or command: name, signature, inputs
  and outputs>
- **Where**: <test file to add or extend, framework already in use>

| Case    | Given                                                 | When                                       | Then                        | Covers |
| ------- | ----------------------------------------------------- | ------------------------------------------ | --------------------------- | ------ |
| nominal | <state>                                               | <action>                                   | <observable result>         | R1     |
| edge    | <state>                                               | <action>                                   | <observable result>         | S3     |
| bound   | <more rows than the bound, e.g. 210 notes on one day> | <the request that hits it, e.g. limit 500> | <the bound, e.g. 200 notes> | R4     |
| refusal | <state>                                               | <action>                                   | <observable result>         | R2     |

<When nothing can be tested first, replace the above with: "Tests: none — <reason>. Guarded
by: <existing check>.">

## How to check

<The command that runs these tests, and for each outcome the observation that shows it.>

## If unsure

Stop and say what is unclear or contradicts the code. Do not choose an interpretation.

## Documentation update

<Only when this ticket carries one: the full draft, to write in the same pull request as
the code. Omit otherwise.>

## Out of scope

- <What this ticket must not touch>
```
