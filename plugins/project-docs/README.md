# project-docs

Scaffold and keep a project's documentation current, at a level of effort proportional to
its size — one `DOCUMENTATION.md` for a spike, a flat `docs/` for a normal project,
a `docs/` tree with numbered ADRs for a big one.

## Skill

| Skill | Use when |
| --- | --- |
| [project-docs](skills/project-docs/SKILL.md) | Setting up docs, or checking them for drift |

## Tooling

[`scripts/docs_index.py`](scripts/docs_index.py) is a deterministic, stdlib-only Python
checker: it regenerates `docs/INDEX.md` and flags documentation that has fallen behind
the code it describes, with zero model calls. Run its test suite with
`python3 tests/docs_index_test.py`.
