# feature-planning

Turn an idea into a rich feature spec and self-contained development tickets:

```
grill  →  to-spec  →  to-tickets
```

Built for setups where the **feature board**, the **ticket tracker** and the **code** are
three independent places (for example Linear for features, GitHub for tickets, a git
clone for code). You name them in your request; there is no configuration file. The
plugin writes nothing to your repository or to disk: drafts live in the conversation,
and published work lives in the board and the tracker. See
[`CONVENTIONS.md`](CONVENTIONS.md) for the rules every skill shares.

## Skills

| Skill                                    | Use when                                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [grill](skills/grill/SKILL.md)           | An idea or need is still fuzzy — interview until it is precise and agreed                         |
| [to-spec](skills/to-spec/SKILL.md)       | The need is agreed and larger than one ticket — publish the rich spec as a feature on the board   |
| [to-tickets](skills/to-tickets/SKILL.md) | The spec is published — slice it into tickets, publish them, then (separately) release them       |

All three are invoked explicitly; none runs on its own.

## Downstream contracts

The tickets feed an automated development system that drops what it cannot read. Each
tracker's rules live in [`contracts/`](contracts/):

- [GitHub](contracts/github.md) — labels, sub-issues, order of creation, what a ticket must contain.

No other tracker has a contract yet. The skills say so and ask for its rules rather than
guessing them from GitHub's.
