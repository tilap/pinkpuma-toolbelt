# Review gates

Three checkpoints between an idea and merged code. Each is a **checklist written for a
reviewer, not for a specific one**: today a person runs it, later an agent can run the
same list unchanged. So every item is a yes/no question, names the **evidence** that
answers it, and says what a "no" triggers. Nothing here asks the reviewer to "use
judgement" without saying on what.

The plugin only runs gate A (it plans). Gates B and C are carried out downstream, on the
pull requests the development system opens; the plugin prepares their inputs so the
reviewer has everything at hand: criteria that can fail in the tickets, and a review sheet
on the feature.

| Gate | When | Reviewer sees | Run by |
| ---- | ---- | ------------- | ------ |
| **A. Plan** | Before the tickets are released | Spec, tickets as published, review sheet | the person, with `to-tickets` |
| **B. Ticket** | Each ticket's pull request, into the feature branch | The ticket body, the diff, the check results | downstream |
| **C. Feature** | The final pull request | The feature body, the review sheet, the cumulative diff | downstream |

## How to report

Whoever runs a gate answers **every item** with `pass`, `fail` or `n/a`, and for each
`fail` or `n/a` one line of evidence. A single `fail` blocks the gate. Do not summarise
("looks fine"): the point of the list is that nothing is skipped. A reviewer that is an
agent returns the same table.

## Gate A — Plan (before release)

Run after the tickets are published and before any ready label is added.

1. **Every rule is covered.** Each business rule `R*` of the spec appears in at least one
   ticket's "Done when". *Evidence*: the review sheet's matrix has no empty rule row.
   *On fail*: add the missing outcome to a ticket, or ask `grill` whether the rule is out.
2. **Every scenario is covered.** Same, for each acceptance scenario `S*`.
3. **No orphan criterion.** Each "Done when" line traces to a rule, a scenario, a
   documentation update, or the project's own checks. *On fail*: the ticket invents
   scope — remove the line or send the user back to `grill`.
4. **Every documentation update lands once.** Each `DU*` is in exactly one ticket, in full.
5. **Cold read.** Reading only the body of ticket N, can you say what to change, how to
   know it is done and what not to touch? *Evidence*: the three answers, written down.
   *On fail*: rewrite the body.
6. **Every criterion can fail.** For each outcome, the check that would show it false is
   named (a case of the ticket's Tests section, a command, an observable state), and it would
   fail on the starting commit.
7. **Green alone.** No ticket needs a later one to compile or pass its checks.
8. **Order is enforced by blocks.** Every required order is a *blocked by* edge, not a
   priority; no cycle.
9. **Cross-cutting concerns decided.** For each of: data migration, security and
   permissions, performance or limits, dependency changes, backward compatibility — either
   a ticket carries an outcome for it, or the spec's Out of scope says why not.
10. **No decision hidden in a ticket.** Nothing in a ticket body answers a question the
    spec lists as open.
11. **Ticket contract.** The tracker contract's checklist passes (labels, links, titles).
12. **Tests are specified first.** Each ticket has a Tests section — cases in Given / When /
    Then covering its rules and scenarios, the interface under test named, the test file and
    framework given — or an explicit `Tests: none` with its reason and the existing check
    that guards it. *Evidence*: per ticket, the cases or the reason. *On fail*: write the
    cases, or state why there are none; a ticket cannot go out without one or the other.

## Gate B — Ticket (each pull request)

1. **Every "Done when" box is met.** Each line is matched to the diff or a test that
   proves it. *Evidence*: line → test or file. *On fail*: send back with the unmet line.
2. **Each outcome has a test that would fail without the change.** A test that passes on
   the starting commit proves nothing. *Evidence*: the test, and why it fails before. The
   tests are those of the ticket's Tests section, written before the code and not edited by
   it; a test changed to pass is a `fail`.
3. **Tests sit at the agreed seams** and check behaviour, not implementation details.
4. **Nothing in "Out of scope" was touched**, and the diff holds no change the ticket
   does not call for. *On fail*: ask to split the extra change out.
5. **The documentation update is written** as drafted, when the ticket carries one.
6. **The project's own checks pass** — and were not weakened (no skipped test, loosened
   assertion, or removed check to get green).
7. **Cross-cutting items in the ticket are done** (migration reversible, permission
   enforced, limit applied, dependency justified), when it lists any.
8. **No guess was made.** Where the ticket was unclear or contradicted the code, the work
   stopped and said so instead of picking an interpretation. A silent choice is a `fail`.

## Gate C — Feature (final pull request)

1. **Traceability.** Every row of the review sheet's matrix points at a merged ticket and a
   test that now exists in the cumulative diff. A rule or scenario with no test is a `fail`.
2. **Scenarios hold together.** Each acceptance scenario `S*`, played end to end at its
   seam on the feature branch, gives the expected result — not only inside its own
   ticket. *Evidence*: the command or test run and its output.
3. **Tickets that did not land are accounted for.** A feature can land without some of its
   tickets: list which are missing and which rule or scenario each leaves uncovered.
4. **No scope creep** across the whole diff against the spec's Out of scope.
5. **Documentation updates** are all written, and the index or entry each names exists.
6. **Cross-feature regression.** The project's full check suite passes on the feature
   branch, not only the per-ticket runs.
7. **The feature body still describes what was built.** If the work diverged, say where.
