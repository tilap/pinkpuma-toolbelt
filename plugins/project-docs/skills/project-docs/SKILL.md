---
name: project-docs
description: >-
  Scaffold, fill, or check a project's documentation — README, AGENTS.md, CHANGELOG,
  and a docs/ set sized to the project. Use when setting up documentation for a project,
  checking whether it has drifted from the code, or asked where the documentation stands.
metadata:
  plugin: project-docs
---

# Project docs

One job: bring this project's documentation to a state that is true and complete, sized
to how big the project actually is. Scaffolding an empty repo and fixing a drifted set
are the same operations from different starting points.

## Step 1 — Read the state, every time

Never assume it, even on a repository that looks obviously empty.

**Shape** — a project uses exactly one, never mixed:

| Found at the root | Shape |
| --- | --- |
| `DOCUMENTATION.md`, no `docs/` | `single-file` |
| `docs/` with subdirectories | `tree` |
| `docs/` with markdown only at the top level | `flat` |
| Neither | undecided — pick one in Step 2 |
| Both `DOCUMENTATION.md` and `docs/` | **stop, report it, ask** — never merge or delete either on your own |

**Inventory** — which owed files exist, which are placeholder-only.

**Checker** — run the deterministic script for the rest (staleness, dead globs, broken
links, missing frontmatter — see [scripts/docs_index.py](scripts/docs_index.py) for the
full list of what it catches):

```bash
python3 scripts/docs_index.py --project . --check --json
```

(`scripts/docs_index.py` sits next to this `SKILL.md`, one level up — resolve the path
relative to wherever this skill's files were loaded from, not relative to the target
project's own working directory.)

On a repository with no documentation, it reports almost everything missing — that's a
valid reading of the state, not a failure of the script.

## Step 2 — Pick or confirm the shape

Default for an empty repository: **`flat`**. On a `partial` repository, infer the shape
already in use from what's on disk rather than imposing the default.

- **`single-file`** — one `DOCUMENTATION.md` at the root, for a script or a spike.
  Sections in this order: `## Purpose`, `## Architecture`, `## Development`,
  `## Decisions`, and `## Glossary` if the project has domain vocabulary worth naming.
  No index — the headings are the index.
- **`flat`** — one file per intention, directly under `docs/`, for a normal project:
  `docs/INDEX.md`, `PRODUCT.md`, `ARCHITECTURE.md`, `DECISIONS.md`, `DEVELOPMENT.md`,
  plus `[TESTING.md]`, `[RUNBOOKS.md]`, `[DATA-MODEL.md]`, `[THREAT-MODEL.md]`,
  `[TROUBLESHOOTING.md]` **on demand only** — an empty one is worse than none.
- **`tree`** — same documents, grouped under `docs/context/`, `docs/architecture/`,
  `docs/decisions/` (numbered `ADR-NNNN-kebab-title.md` + a `README.md` index),
  `docs/operations/` — for a project big enough to want numbered decisions or several
  architecture documents.

A project that has outgrown its shape (a `single-file` project needing a runbook, a
`flat` project needing numbered ADRs) is told so — never silently migrated.

Root trio in every shape: `README.md`, `AGENTS.md`, `CHANGELOG.md` (Keep a Changelog
format, live `## [Unreleased]` section).

## Step 3 — Show the plan, then act

Print what's missing and what's next, ask once for the whole plan, then run in order —
stop at the first step that needs an answer only the user can give.

1. **Intake** — infer name, goal, stack, commands from `package.json` / `go.mod` /
   `Cargo.toml` / `pyproject.toml` / existing `README.md` / source layout. Ask only what
   can't be inferred (project name, one-sentence goal) — one line each, no elaboration.
2. **Create missing files only.** Never replace an existing file wholesale; merge in
   missing sections when a file exists but is incomplete.
3. **Frontmatter** on every file under `docs/` (or once at the top of `DOCUMENTATION.md`
   in `single-file`): `title`, `summary` (one line, what the doc answers), `status`
   (`draft` | `verified` | `stale`), `last_verified` (`YYYY-MM-DD`, **never advance
   without actually reading the doc against the code**), `covers` (globs of the code it
   describes, `[]` if none). Freshly scaffolded ⇒ `status: draft`, today's date.
4. **Judge existing documents, don't just fill gaps.** A complete doc set can be
   entirely wrong — a confidently wrong `ARCHITECTURE.md` is worse than a missing one.
   For every document the checker flags as presumed-stale: read it against the code its
   `covers` globs point at, decide `current` / `stale` / `wrong`, and either fix it or
   mark it `stale` and say so. Never bump `last_verified` on a document nobody read.
5. **Regenerate the index** (`flat`/`tree` only — `single-file` has none):
   ```bash
   python3 scripts/docs_index.py --project . --write
   python3 scripts/docs_index.py --project . --check
   ```

## What this covers, and what it does not

- **Subject**: product and technical documentation only. Never `LICENSE`,
  `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, or a root `SECURITY.md` — leave them alone,
  never create them, never report them as missing. Internal security design belongs in
  `THREAT-MODEL.md`, not `SECURITY.md`.
- **Surface**: the project's own files — never `.claude/`, `.cursor/`, `.agent/`, or any
  plugin's own `skills/`/`agents`/`commands` as "this project's documentation".
- **Time**: the present, plus decided history. Roadmaps belong in the issue tracker, not
  in `docs/`.
- Ask: "would this sentence be true if a human had written it, today?"

## Constraints

- **Idempotent** — safe to re-run; minimal diff every time.
- **No forced migration.** A repository documenting itself differently already wins —
  report the divergence once, follow it, never move files to match this skill's
  structure without being asked.
- **Never invent a project fact.** Ask, or leave the placeholder visible and say so in
  the report.
- Do not commit changes unless the user asks.

## Report, in this order

1. Presumed-stale documents first (with day counts) — the finding a human wouldn't
   catch by skimming.
2. Structural findings (missing frontmatter, dead `covers` globs, broken links).
3. Created / merged / skipped (already present) / left unfilled, and why.
4. The single next useful step — usually the architecture document, since it unblocks
   every later session.
