# spec-workflow

Turn a business need into GitHub issues an agent can build from, in three steps:

```
to-brief  →  to-spec  →  to-tickets
```

Self-contained: no other plugin and no glossary file required. It reads the project's
`docs/context/`, `docs/architecture/`, `docs/decisions/` and `docs/operations/` when they
exist, and leaves the repository untouched: working files live in `.agent/specs/<slug>/`
(excluded through `.git/info/exclude`), and the ADRs and doc updates a need calls for
are drafted into the tickets for implementation to write. The chain ends once the
issues are on GitHub. See [`CONVENTIONS.md`](CONVENTIONS.md) for the rules every skill
here shares.

## Skills

| Skill                                    | Use when                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------ |
| [to-brief](skills/to-brief/SKILL.md)     | A business need is still fuzzy — interview until it is agreed, draft ADRs      |
| [to-spec](skills/to-spec/SKILL.md)       | The brief is agreed and spans several sessions — publish the parent spec issue |
| [to-tickets](skills/to-tickets/SKILL.md) | The spec is published — slice it into vertical, blocking-linked sub-issues     |

All three are invoked explicitly; none runs on its own. Publishing to GitHub needs the
`gh` CLI, authenticated, inside a clone with a GitHub remote.
