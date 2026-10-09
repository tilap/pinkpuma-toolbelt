---
name: to-spec
description: >-
  Synthesize an agreed brief (or the current conversation) into a technical spec and
  publish it as the parent GitHub issue — no new interview, only what was already
  decided, plus the agreed test seams. Use after to-brief when the need spans several
  working sessions and must later be split into tickets.
disable-model-invocation: true
metadata:
  plugin: spec-workflow
---

# To spec

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md) — read it first (chain, working
files, documentation layout, GitHub, writing rules). The repository stays untouched:
the only local file this skill writes is `.agent/specs/<slug>/spec.md`.

One job: turn what was decided into the document a fresh session builds from, and
publish it as the parent issue the tickets will hang off. **Do not interview the user.**
The spec records decisions; it is not a place to make new ones. If it states something
nobody decided, that is a defect.

## Step 1 — Load the source

In order of preference:

1. A brief path or slug passed as argument, or `.agent/specs/<slug>/brief.md` for the
   need discussed in this conversation.
2. An issue reference passed as argument — read its body and comments.
3. The current conversation.

If the brief is still `Status: draft`, or an open question blocks a decision the spec
needs, **stop**: list the blocking questions and recommend finishing `to-brief`. Do not
fill the gap yourself.

If the need fits one working session, say so and recommend going straight to
`to-tickets` for a single ticket; continue only if the user insists.

## Step 2 — Ground it in the project

Read `docs/architecture/`, the ADRs in the area, `docs/operations/` when the need has a
runtime or deployment side, and the code the need touches. The spec uses the project's
vocabulary and module names, and respects accepted ADRs — or names the one it reopens.

## Step 3 — Agree the test seams

The one exchange this skill has with the user. A **seam** is the boundary at which the
feature's behaviour is tested from outside (a public API, a command, a module interface,
a UI route).

- Prefer existing seams to new ones, and the highest seam that can observe the
  behaviour. The ideal number is one.
- If a new seam is needed, propose it at the highest point possible.

Present the seams as a short list — seam, what it observes, existing or new — and wait
for the user to confirm or correct them. Implementation and review will hold the work to
these seams, so this is where a wrong one is cheapest to catch.

## Step 4 — Draft

Fill the template below. Carry business rules and scenarios over from the brief with
their identifiers (`R1`, `S2`) and wording intact; do not soften a precise rule into
prose. Carry the brief's documentation update drafts (`DU1`, …) over in full — they are
work for implementation, not something this skill writes. If the work is purely technical (a refactor, a module boundary), replace User
stories with "Technical change — no user-facing stories" and put the weight on
Implementation and Testing decisions.

## Step 5 — Publish

1. Search GitHub for overlapping open issues; show any hit.
2. Show the full draft and the title. Publish only on explicit approval.
3. Write the body to `.agent/specs/<slug>/spec.md`, create the issue with
   `--body-file` and the `spec` label (see Conventions for labels), then put
   `Issue: #<n>` as the first line of the local copy.
4. Report the issue link and recommend `to-tickets` **in this same conversation**, so the
   spec never has to be fetched back.

The published spec is a snapshot of what was known today. When implementation teaches
something that must last, it belongs in an ADR or `docs/context/`, not in an edited spec.

## Spec template

```markdown
## Problem

<From the user's perspective — from the brief.>

## Solution

<What will be true for the user when this ships — from their perspective.>

## User stories

1. As a <actor>, I want <capability>, so that <benefit>.
2. …

<Extensive: cover every actor and every scenario of the brief.>

## Business rules

- **R1** — <verbatim from the brief>

## Acceptance scenarios

- **S1 — <name>** (R1): Given …, when …, then ….

## Implementation decisions

- <Modules built or changed, and their interface changes>
- <Data / schema changes>
- <API or event contracts>
- <Technical decisions taken in the brief, with their reason>

## Testing decisions

- **Seams**: <agreed seams, and what each observes>
- Test external behaviour at those seams, not implementation details.
- **Prior art**: <existing tests of the same kind to follow>

## Constraints

- <ADR or document title> — <link> — <what it imposes here>

## Documentation updates

<Every DU draft from the brief, in full, for implementation to write. "None" if none.>

## Out of scope

- <Every refusal from the brief, plus anything adjacent a builder might gold-plate>

## Open questions

- <Only questions the user explicitly deferred; never one this spec silently answers>

## Source

<Brief path or issue reference, and date.>
```
