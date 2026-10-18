---
name: to-spec
description: >-
  Synthesize what was decided (the grill conversation, or a pasted or linked source) into a
  rich, complete feature spec and publish it as a feature on the feature board — no new
  interview, only what was already decided. Use after grill, when the need is larger than a
  single ticket and must later be split into tickets.
disable-model-invocation: true
metadata:
  plugin: feature-planning
---

# To spec

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md) — read it first (the three
places, nothing written to disk, publishing and writing rules). Contract for the board's
system, when it has one: [../../contracts/](../../contracts/).

One job: turn what was decided into the complete document a fresh reader builds from, and
publish it as a **feature** on the board. **Do not interview the user.** The spec records
decisions; it is not where new ones are made. If it states something nobody decided, that
is a defect.

## Step 1 — Load the source

In order of preference: the **Settled so far** of `grill` in this conversation; a feature,
issue or document the user points at (read its body and comments); the conversation itself.

If the understanding is not agreed, or an open question blocks a decision the spec needs,
**stop**: list the blocking questions and recommend finishing `grill`. Do not fill the gap.
If the need fits a single ticket, say so and recommend `to-tickets` directly; continue only
if the user insists.

## Step 2 — Ground it in the project

Read the architecture notes, decision records in the area, and the code the need touches
(read-only). The spec uses the project's vocabulary and module names and respects accepted
ADRs — or names the one it reopens.

## Step 3 — Agree the test boundaries

The one exchange this skill has with the user. A **seam** is the boundary at which the
feature's behaviour is tested from outside (a public API, a command, a module interface, a
UI route). Prefer existing seams to new ones and the highest one that can observe the
behaviour; the ideal count is one. Present them as a short list — seam, what it observes,
existing or new — and wait for confirmation or correction. Tickets will hold their
outcomes to these seams, so this is where a wrong one is cheapest to catch.

## Step 4 — Draft

Fill the template below, **rich and complete**: this is the single place the full reasoning
lives. Carry business rules and scenarios with their identifiers (`R1`, `S2`) and wording
intact; do not soften a precise rule into prose. Include documentation-update drafts in
full. Use **no file paths** (see Conventions). For a purely technical change (refactor,
module boundary), replace User stories with "Technical change — no user-facing stories" and
put the weight on Implementation and Testing decisions.

If the board has a contract file in [../../contracts/](../../contracts/), apply what it says
about the feature. On GitHub the body is read word for word as the statement the final
reviewer sees, so it leads with the problem and the solution. No contract file for the
board in use → say so; the spec is still drafted.

## Step 5 — Publish

1. Settle the board-side fields with the user in one line each: project, priority, blocking
   features. Apply only what exists (see Conventions for labels).
2. Search the board for overlapping features; show any hit.
3. Show the full draft and the title — a real title, not a placeholder. Publish only on
   explicit approval.
4. Create the feature through the tool for that board (`gh issue create --body-file -` for
   GitHub; the connector for Linear or Jira; otherwise hand the text over for pasting).
   **Do not add the feature-ready label** (GitHub default: `feature-ready`) or any label that tells a downstream system to start: that
   is the last step of `to-tickets`, once its tickets exist.
5. Report the feature's link and tell the person to type `/feature-planning:to-tickets` **in
   this same conversation**, so the spec never has to be fetched back (Handing over to the
   next step, in the conventions). You cannot start it yourself.

The published spec is a snapshot of what was known today. Lasting truths discovered during
implementation belong in a decision record or the project's docs, not in an edited spec.

## Spec template

```markdown
## Problem

<From the user's perspective.>

## Solution

<What will be true for the user when this ships — from their perspective.>

## User stories

1. As a <actor>, I want <capability>, so that <benefit>.
2. …

<Extensive: cover every actor and every scenario.>

## Business rules

- **R1** — <precise rule, with its numbers and limits>

## Acceptance scenarios

- **S1 — <name>** (R1): Given …, when …, then ….

## Implementation decisions

- <Modules built or changed, and their interface changes>
- <Data / schema changes>
- <API or event contracts>
- <Technical decisions taken, with their reason>

## Testing decisions

- **Seams**: <agreed seams, and what each observes>
- Test external behaviour at those seams, not implementation details.
- **Prior art**: <existing tests of the same kind to follow>

## Constraints

- <ADR or document> — <what it imposes here>
- **Cross-cutting**: <migration, security and permissions, performance or limits,
  dependencies, backward compatibility — each with its decision, or "not concerned">.
  Tickets turn these into outcomes and reviewers check them, so none is left implicit.

## Documentation updates

<Each draft in full, for implementation to write — nothing is written yet. "None" if none.>

## Out of scope

- <Every refusal, plus anything adjacent a builder might gold-plate>

## Open questions

- <Only questions the user explicitly deferred; never one this spec silently answers>
```
