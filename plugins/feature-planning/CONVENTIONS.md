# Feature planning conventions

Shared by every skill in this plugin. Read this once; each `SKILL.md` links back here
instead of repeating it.

## The chain

```
grill  →  to-spec  →  to-tickets  →  (release to the dev system, then development)
```

| Skill        | Decides? | Output                                                         | Lives in               |
| ------------ | -------- | -------------------------------------------------------------- | ---------------------- |
| `grill`      | Yes      | A shared, complete understanding of the need                   | The conversation       |
| `to-spec`    | No       | A rich feature spec, published                                 | The feature board      |
| `to-tickets` | No       | Self-contained, dependency-linked tickets, published, then released | The ticket tracker |

Only `grill` makes decisions. `to-spec` and `to-tickets` record and slice what was already
decided: if they need a decision nobody made, they stop and send the user back to `grill`
rather than inventing one.

Skip a step that buys nothing: a need small enough for one ticket can go from `grill`
straight to `to-tickets`.

## The three places

Work involves up to three independent systems. They may be the same product or three
different ones (feature board on Linear, tickets on GitHub, code in a git clone).

| Place             | What it holds                         | Examples                              |
| ----------------- | ------------------------------------- | ------------------------------------- |
| **Feature board** | Features and their specs              | GitHub issues, Linear, Jira, GitLab   |
| **Ticket tracker**| Tickets, the units of development work| GitHub issues, Jira, GitLab           |
| **Code**          | The source, read-only for these skills| A git clone — usually the current directory |

The user names them in their request ("board: Linear project X, tickets: GitHub org/repo").
There is **no configuration file** and nothing is remembered between sessions.

- Use what the user said. For what they did not say, propose one inference in a single
  line and let them correct it: code = the current directory, GitHub repository = its git
  remote. Ask only when nothing can be inferred.
- Reach each system with the tool at hand: the `gh` CLI for GitHub, a connected MCP
  connector for Linear or Jira. If no tool reaches a system, produce the text and tell the
  user where to paste it — never pretend something was published.
- Features and tickets in different systems are linked by reference (the feature's URL),
  not by a native parent link.

### Downstream contracts

The tickets are consumed by an automated development system whose expectations are not
negotiable: a ticket it cannot read is dropped silently. Each tracker has a contract file
under [contracts/](contracts/) describing what that system reads and requires:

- GitHub: [contracts/github.md](contracts/github.md).
- Any other system: **no contract exists yet.** Say so, do not guess one from GitHub's, and
  ask the user for its rules before publishing tickets there. Drafting is still fine.

## Nothing is written to the repository or to disk

These skills prepare work; they never do it. They create no file anywhere — not in the
repository, not under `docs/`, not in `.git/`, not as a scratch copy. Working state lives
in the conversation; durable state lives in the board and the tracker.

Consequences:

- `grill` keeps a visible running list of settled decisions after each round, so the
  conversation itself is the record.
- `grill`, `to-spec` and `to-tickets` are meant to run in **one conversation**. If it was
  lost, restart from the published spec (read it back from the board), not from memory.

## Reading the project

Before asking or drafting, read what the project already settled — read-only:

- `README.md`, `AGENTS.md`/`CLAUDE.md` — stack, commands, conventions.
- Whatever documentation exists (`docs/`, architecture notes, decision records, glossary).
- The code of the area the need touches.

Never block on a missing document and never create one. Treat a document marked stale as a
hint to verify against the code.

**Facts are your job, not the user's.** Anything the docs or code can answer — how something
works today, what a module is called — is looked up, never asked.

### Vocabulary

Write every artifact in the words the project's docs and code already use for domain
concepts. When the user uses another word for an existing concept, say so and use the
project's. When a concept has no name yet, `grill` settles one.

### Decision records

If anything produced contradicts an accepted decision record (ADR), say so explicitly
rather than overriding it silently: _Contradicts ADR-0007 — reopening it because …_

## Documentation updates are drafted, not written

A need may call for a lasting documentation change: a decision record, a business rule.
These skills **draft** them; they never write them. The draft travels in the spec and lands
in the one ticket that makes it true, to be written in the same pull request as the code.

Propose a decision record only when all three hold: hard to reverse; surprising without
context; a real trade-off with genuine alternatives. Most needs produce none. Leave its
number as `ADR-NNNN` for the implementer, and match the project's own record format.

## Publishing rules

- **Nothing is published without the user's explicit approval of the exact content.** An
  approval covers one action, once; it does not carry over to the next.
- Search for overlapping existing items first; show any hit before creating.
- Never close, retitle or rewrite an item the user did not ask you to change.
- Apply a label only if it already exists; if it does not, ask before creating it.
- Report what was done and what was not. Never claim a publication that did not happen.

## Writing rules for everything published

- **Language**: that of the project's existing documentation; if none, the conversation's.
- **Never invent.** Every statement traces to the conversation, the docs or the code. A gap
  goes into an "Open questions" list, visibly. Precise answers stay precise: a number, an
  ordering or an explicit "must not" is copied as-is.
- **Paths.** The spec uses none: they go stale before work starts, so it names modules,
  interfaces and behaviours. **Tickets may use them**, because the agent that builds a
  ticket sees only the repository: a path that saves it a search is welcome. Keep to the
  few that matter (where to change, which test to extend); no line numbers; no code except
  a short snippet that encodes a decision better than prose.
- **Behaviour, not procedure** in the spec: what the system must do, not which function to edit.
