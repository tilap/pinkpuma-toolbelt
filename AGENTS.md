# AGENTS.md

Instructions for AI coding agents working in this repository. For a human-facing
overview, see [README.md](README.md); for the full guide to adding or changing a plugin
(create / change / remove / release, including local clone installs), see
[HOWTO.md](HOWTO.md). This file is the short, load-bearing version — read it before
making any change under `plugins/`.

## What this repo is

A marketplace of Claude Code / Cursor plugins. Each plugin lives in `plugins/<name>/` and
is listed in `.claude-plugin/marketplace.json`. Every plugin follows the same layout —
see [HOWTO.md § Repository structure](HOWTO.md#repository-structure) before creating or
restructuring one.

## Hard rules

1. **Uniform plugin layout.** Every plugin has `.claude-plugin/plugin.json` and
   `.cursor-plugin/plugin.json` with matching `name`, `version`, `description`, and
   `author`, a `skills/` directory, and a `README.md`. Don't give one plugin a one-off
   structure without updating HOWTO.md to match — the docs and the repo must not drift
   apart.

2. **Validate before every commit that touches a plugin:**

   ```bash
   claude plugin validate --strict ./plugins/<name>
   ```

   Zero warnings, not just zero errors. It's free and instant — never skip it.

3. **Run the test suite when touching `project-docs`'s script:**

   ```bash
   python3 plugins/project-docs/tests/docs_index_test.py
   ```

   It must pass before committing any change to
   `plugins/project-docs/scripts/docs_index.py`.

4. **Keep skills portable by default.** Follow
   [HOWTO.md § Write a skill or an agent](HOWTO.md#write-a-skill-or-an-agent) for which
   frontmatter fields are safe on both Claude Code and Cursor. If a change genuinely needs
   a host-specific field, say so in the skill's own body so it degrades on the other host
   instead of silently doing less than advertised.

5. **A new plugin, or a registration change, updates
   `.claude-plugin/marketplace.json`.** A plugin that exists on disk but isn't listed
   there doesn't ship.

6. **Before adding or changing a skill or agent**, follow
   [HOWTO.md § Before adding or changing a skill or agent](HOWTO.md#before-adding-or-changing-a-skill-or-agent):
   check the current docs for that component type, check whether it already exists
   elsewhere, and use the available generation/review/eval tooling rather than
   hand-writing everything from scratch.

7. **All plugin content is English** — `SKILL.md` and agent bodies, frontmatter
   descriptions, `README.md`, code comments. No exceptions for a quick draft.

8. **One focused commit per logical change.** A new plugin, or a change confined to one
   plugin, gets its own commit. Don't bundle unrelated plugins or unrelated fixes
   together. When committing changes to this repository itself, follow the conventions in
   [`plugins/git-workflow/CONVENTIONS.md`](plugins/git-workflow/CONVENTIONS.md).

## Quick reference

| Task                  | Command                                                                      |
| --------------------- | ---------------------------------------------------------------------------- |
| Validate one plugin   | `claude plugin validate --strict ./plugins/<name>`                           |
| Validate every plugin | repeat the above for `git-workflow`, `handoff`, `project-docs`, `dev-agents` |
| Test the docs checker | `python3 plugins/project-docs/tests/docs_index_test.py`                      |

## Definition of done for any change under `plugins/`

- [ ] The touched plugin's `.claude-plugin/plugin.json` and `.cursor-plugin/plugin.json` still match
- [ ] `claude plugin validate --strict` passes with no warnings for every plugin touched
- [ ] `docs_index_test.py` passes, if `project-docs`'s script changed
- [ ] `.claude-plugin/marketplace.json` reflects any plugin added, renamed, or removed
- [ ] HOWTO.md updated if the change alters the repo's structure or conventions, not just one plugin's content
