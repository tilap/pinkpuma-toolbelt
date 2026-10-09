---
name: to-brief
description: >-
  Turn a raw business need into an agreed brief through a relentless, round-by-round
  interview grounded in the project's docs and code, drafting the ADRs and doc updates
  it calls for without touching the repository. Use when the user brings a feature
  request or business need that is still fuzzy, before any spec or ticket is written.
disable-model-invocation: true
metadata:
  plugin: spec-workflow
---

# To brief

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md) — read it first (chain, working
files, documentation layout, ADR format, writing rules). The repository stays untouched:
the only file this skill writes is the brief.

One job: interview the user until you and they share one precise understanding of a
business need, then write it down as a brief that a fresh session can build on without
the user in the room. This is the only skill in the chain that makes decisions; the
next ones only record and slice them.

## Step 1 — Take in the need

Start from the user's argument or the conversation. If they point at an issue, read its
body and comments. Restate the need in two sentences — who has the problem, and what
they cannot do today — and pick the `<slug>` for `.agent/specs/<slug>/`.

If `.agent/specs/<slug>/brief.md` already exists, resume it: its Decisions are settled,
its Open questions are the starting frontier.

## Step 2 — Gather the facts yourself

Read the project documentation that touches the need (see Conventions) and the code in
that area. **Facts are your job, never the user's**: anything the docs or code can
answer — how something works today, what a module is called, which ADR applies — you
look up instead of asking. If the host lets you run an exploration in the background,
do so and keep interviewing on the questions that do not depend on it.

## Step 3 — Interview in rounds

Map the need as a **decision tree**: every decision branches into the decisions that
depend on it. The **frontier** is every open decision whose prerequisites are settled —
what you can ask now without guessing at answers you have not heard yet.

Each round, ask the frontier — at most the 7 questions that unblock the most, if it is
larger — then wait for the answers. Number every question, give your recommended answer,
and word it so that "yes" accepts the recommendation:

```
❓ **Q1 — <title>**: <question, with options when there are several>

➡️ <recommended answer, and why in one line>
```

A question whose answer depends on another question still open this round belongs to a
later round. Each answer reshapes the tree: recompute the frontier and ask the next
round.

Work the tree **business first**, roughly in this order — skip any level the need
already makes obvious:

1. **Problem and outcome** — who suffers today, what success looks like and how anyone
   would observe it.
2. **Actors** — who acts, who is affected, who is allowed to do what.
3. **Business rules** — the invariants, limits, states and transitions; precise numbers,
   not "a few" or "quickly".
4. **Scenarios** — concrete examples, then edge cases you invent to probe the boundaries
   of each rule: empty, concurrent, repeated, cancelled, partial, unauthorised, at the
   limit.
5. **Scope** — what is deliberately refused. Explicit "no"s are often the most useful
   lines of the brief.
6. **Technical shape** — only the choices that are visible to the business or hard to
   reverse. Leave the rest to implementation.

While interviewing, keep the language sharp:

- **Vocabulary** — when the user uses a word that the docs or code use differently, or
  two words for one concept, stop and settle it: "The docs call this a *Booking*; you
  said *reservation* — same thing?" Use the project's word from then on.
- **Code disagreements** — when the user states how something works and the code says
  otherwise, show the contradiction and ask which one is right.
- **ADR conflicts** — when an answer contradicts an accepted ADR, say so and ask whether
  the ADR is being reopened.

## Step 4 — Record as things settle, not at the end

After every round, rewrite `.agent/specs/<slug>/brief.md` (template below) with what is
now settled, keeping `Status: draft`. A cleared or crashed session then loses nothing.
Before the first write, make sure `.agent/specs/` is in `.git/info/exclude`.

When a decision passes the three ADR gates in Conventions, propose the ADR in the next
round (as a question, with the drafted title and decision). On a yes, add its full draft
to the brief's Documentation updates. Do the same for a lasting business rule or term
that `docs/context/` contradicts or lacks: propose the exact edit, and on a yes draft it
in the brief. Never write either into `docs/` — implementation does that.

Every other decision lives in the brief's Decisions list. Nothing decided in this
session may exist only in the conversation.

## Step 5 — Close

The interview is done when the frontier is empty: every branch visited, nothing silently
assumed, Open questions empty or explicitly deferred by the user. Then:

1. Show the brief and ask the user to confirm it matches their understanding.
2. On confirmation, set `Status: agreed` and write the file.
3. Size it, and end with exactly one recommended next step:
   - fits one working session → skip `to-spec`; run `to-tickets`, which turns the brief
     into a single ticket;
   - spans several sessions → run `to-spec` in this same conversation.

## Brief template

```markdown
# Brief — <need, in the project's words>

Slug: <slug>
Date: <YYYY-MM-DD>
Status: draft | agreed
Source: <issue reference, or "conversation">

## Problem

<Who has the problem, what they cannot do today, why it matters. User's perspective.>

## Outcome

<What changes for them when this is done, and how anyone will observe it.>

## Actors

- **<Actor>** — <who they are, what they may do here>

## Vocabulary

<Only terms this need introduces or clarifies; omit the section otherwise.>

- **<Term>** — <one-sentence definition>. _Not_: <rejected synonyms>

## Business rules

- **R1** — <precise rule, with its numbers and limits>
- **R2** — …

## Scenarios

- **S1 — <name>** (R1): Given <state>, when <action>, then <observable result>.
- **S2 — <edge case>** (R1, R2): …

## Out of scope

- <What was deliberately refused, and why if it is not obvious>

## Constraints

- <Constraint> — <link to the ADR or document it comes from>

## Decisions

- **D1** — <decision> — <why>

## Documentation updates

<Drafts for implementation to write — nothing here is in docs/ yet. "None" if none.>

### DU1 — ADR: <title>

<Full ADR draft, in the format from Conventions, number left as ADR-NNNN.
Supersedes: <ADR, if any>.>

### DU2 — docs/context/<file>: <section>

<The exact text to add or replace, and where.>

## Open questions

- <Question — who can answer it. Empty when Status is agreed, unless explicitly deferred.>

## Sizing

<One session or several, in one line, and the recommended next step.>
```
