# HOWTO

How this repository is organized, how to add or change a plugin, and the rules that keep
every plugin working on both Claude Code and Cursor.

## Repository structure

One repository, one marketplace, several plugins:

```
.
├── .claude-plugin/
│   └── marketplace.json          # lists every plugin, by path, for Claude Code
├── plugins/
│   └── <plugin-name>/
│       ├── .claude-plugin/plugin.json   # Claude Code manifest
│       ├── .cursor-plugin/plugin.json   # Cursor manifest — same metadata, separate file
│       ├── skills/
│       │   └── <skill-name>/SKILL.md
│       ├── agents/                      # only for plugins that ship subagents
│       │   └── <agent-name>.md
│       └── README.md                    # what the plugin does, one line per skill/agent
└── README.md
```

Every plugin follows this exact layout. Both manifests point at the same `skills/` (and
`agents/`) directory — they're two views of one set of files, not two copies to keep in
sync by hand.

## Two levels: marketplace repo vs plugin

| Level                              | What it is                       | Versioned how                                                                                |
| ---------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------- |
| Marketplace repository (this repo) | Git container that lists plugins | Ordinary commits and pushes — no product version of the whole repo                           |
| One plugin under `plugins/<name>/` | What users install               | `version` in both manifests + that plugin’s `CHANGELOG.md` + optional tag `<name>-v<semver>` |

You publish **plugins**, not a single version of the marketplace. Bump each plugin’s
manifest `version`, keep a per-plugin `CHANGELOG.md`, and optionally tag
`<name>-v<semver>` when you want a pin-able release.

## Develop against a local clone

Clone this repository, then install plugins **from that working tree** so the agent loads
the skills you are editing.

### Claude Code

```bash
cd /path/to/this-repo
claude plugin marketplace add ./
claude plugin install git-workflow@pinkpuma-toolbelt
# plus any other plugins you need while developing
```

A marketplace registered as `./` reads files from the clone. After editing a `SKILL.md`,
the next invocation usually picks up the change. If it does not, refresh with
`claude plugin marketplace update` or reinstall the plugin.

For day-to-day *use* of this marketplace on other machines or projects, prefer the GitHub
source (see [Distribution](#distribution) and the root [README](README.md#install)). Use
the local `./` marketplace when you are **authoring** plugins in this repo.

### Cursor

Point Cursor’s plugin manager at `plugins/<name>/` inside the clone for each plugin you
need while developing (or link them under `~/.cursor/plugins/local/`). Reload the window
if a skill change does not appear.

Do **not** copy skills into `~/.cursor/skills/` for marketplace development — that
duplicates the source and drifts. One working tree is the source of truth.

## Create a plugin

1. Branch: `feat/add-<name>-plugin`.
2. Scaffold `plugins/<name>/` with `.claude-plugin/plugin.json`,
   `.cursor-plugin/plugin.json`, a `skills/` directory, and `README.md` — layout above.
   Keep `name`, `version`, `description`, and `author` identical in both manifests.
3. Add an entry to `.claude-plugin/marketplace.json` with `source` pointing at
   `./plugins/<name>`. A plugin on disk that is not listed there does not ship.
4. Add a row to the root [README](README.md) Plugins table.
5. `claude plugin validate --strict ./plugins/<name>` — zero warnings.
6. Install the new plugin from the local marketplace and exercise its skills.
7. Commit (prefer `git-workflow` when available).
8. Cut the first release: bump both manifests (typically `0.1.0`), add
   `plugins/<name>/CHANGELOG.md`, validate again.
9. Push. Create the annotated tag `<name>-v<semver>` only when you want installers to pin
   that release.

## Change a plugin

1. Branch for the change.
2. Edit under `plugins/<name>/`. With a local marketplace install, re-test after edits;
   refresh the install if the agent still sees old skill text.
3. `claude plugin validate --strict ./plugins/<name>`. If you touch
   `project-docs`’s script, also run `python3 plugins/project-docs/tests/docs_index_test.py`.
4. Commit.
5. When you want a **consumable** release (not necessarily every commit): bump semver in
   both manifests, cut the changelog, add optional `MIGRATION-<version>.md` if breaking,
   then validate again.
6. Push. Tag `<name>-v<semver>` when consumers should pin the new version.
7. Downstream machines already using the GitHub marketplace update or reinstall that
   plugin (or bump their lockfile pin if they use one).

## Remove a plugin

1. Branch: `chore/remove-<name>-plugin`.
2. Prefer documenting deprecation in a final release when others may still depend on it;
   otherwise remove hard:
   - delete the entry from `.claude-plugin/marketplace.json`
   - remove the root README (and any AGENTS) listing
   - delete `plugins/<name>/`
3. Uninstall locally (`claude plugin uninstall <name>@pinkpuma-toolbelt`, remove any Cursor
   local link).
4. Commit and push. Call out the removal in the PR so consumers know to uninstall.

## Write a skill or an agent

### Keep the portable core to the portable core

Claude Code and Cursor both implement the open *Agent Skills* standard, but each adds its
own extensions on top. Stick to the fields both read when a skill needs to work
identically on either host:

| Field                                                                                                                                                 | Portable?                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `name`, `description`, `metadata`                                                                                                                     | Yes                                                                                |
| `disable-model-invocation`                                                                                                                            | Yes                                                                                |
| `paths` (glob restriction)                                                                                                                            | Yes                                                                                |
| `allowed-tools`, `context: fork`, `arguments`/`$ARGUMENTS`, inline `!command` injection, `model`, `effort`, `shell`, `hooks` in a skill's frontmatter | Claude Code only — Cursor ignores them or shows the literal text, it doesn't error |
| `icon`, `color`                                                                                                                                       | Cursor only — harmless if a skill sets them anyway                                 |

For agents (`agents/*.md`), the portable core is `name`, `description`, `tools`,
`disallowedTools`, `model`. Claude Code's `permissionMode`, `maxTurns`, `memory`,
`isolation: worktree`, and `mcpServers` have no confirmed Cursor equivalent — an agent can
still use them, but write its instructions so it degrades to a fresh, memory-less run
rather than assuming they took effect.

Never name a host-specific tool inside a skill's instructions (e.g. a specific
orchestration command). Describe the strategy in host-neutral prose instead, and let
whichever tool runs the skill route to its own native mechanism.

### Other rules

- Keep a `SKILL.md` body under ~500 lines. Move detail into a sibling reference file and
  link to it, rather than inlining everything.
- The `description` field is what triggers auto-invocation — it matters more than the
  body. Write it as "what this does, use when X", and re-read it cold before committing.
- Set `disable-model-invocation: true` on any skill with a side effect (writes a file,
  commits, modifies state) so it only runs when asked for explicitly.
- A shared convention used by several skills in one plugin belongs in one file at the
  plugin root (e.g. `CONVENTIONS.md`), referenced by each skill — never duplicated.

## Before adding or changing a skill or agent

Three checks, in order, before writing the first line:

1. **Read the current docs for this component type, on both platforms.** APIs and
   conventions move fast on both sides; don't rely on memory or on this file staying
   current forever. Check the official Claude Code docs and Cursor docs for the specific
   component (skill, agent, hook) being touched.
2. **Check whether this already exists, done well, somewhere else.** Search recent
   sources — official changelogs and blogs, the `agentskills.io` registry, well-maintained
   community projects — before building something from scratch. A bundled or
   already-maintained equivalent is worth adopting or pointing to instead of duplicating.
3. **Use the available tooling rather than hand-writing everything**:
   - Draft or iterate a skill with the `skill-creator` tooling available in Claude Code —
     it also helps tune a `description` for reliable auto-invocation, the easiest thing to
     get subtly wrong by hand.
   - Review a first draft the way any other change gets reviewed, and run a security pass
     if it adds a hook or a shell script.
   - Test behavior, not just shape: `claude plugin eval` (or a skill's own eval loop) runs
     real scenarios against the skill; `claude plugin validate --strict` checks the
     manifest for free before every commit.
   - Periodically check `/skill-doctor` (or equivalent) to see which skills are actually
     being used, and retire the ones that aren't.

## Verification tooling reference

| Tool                                      | Checks                                                                           | Cost                                                              |
| ----------------------------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `claude plugin validate [--strict]`       | Manifest schema, required fields, paths                                          | Free, instant — run before every commit                           |
| `claude plugin eval`                      | Behavior against realistic prompts and graders, compared to a no-plugin baseline | Real model calls — run at meaningful milestones, not every commit |
| A skill's own eval loop (where available) | Fast iteration on one skill in isolation                                         | Cheaper, real model calls                                         |
| `/skill-doctor` (or equivalent)           | Per-skill context cost and actual usage frequency                                | Free — run occasionally to prune unused skills                    |

None of these check house style on their own (frontmatter completeness, naming
consistency across plugins, description quality). Where that matters, write a small,
deterministic, stdlib-only checker script rather than an LLM-based one — it costs nothing
to run and catches drift mechanically.

## Distribution

- Register this repository from its GitHub source rather than a local directory path
  (`{"source": "github", "repo": "tilap/pinkpuma-toolbelt"}` in `extraKnownMarketplaces`, or the
  equivalent `claude plugin marketplace add tilap/pinkpuma-toolbelt`). A GitHub source works
  from any machine with git access to the repo; a local directory path only works on the
  machine it was added from.
- This repository is private. Claude Code has no credentials of its own — it clones with
  whatever `git` access the machine already has (an SSH key in `ssh-agent`, or a stored
  HTTPS credential via `gh auth login` + `gh auth setup-git`). Anyone who needs access
  must be added as a collaborator on the repository itself; there's no separate
  permission layer on top of normal GitHub access.
- To scope plugins to one project instead of every project on a machine, install with
  `--scope project` and commit the `.claude/settings.json` it writes (see the root
  [README](README.md#install)). Project-local, personal overrides go in
  `.claude/settings.local.json` instead, which stays out of version control.
- Publishing a plugin to npm is possible for a single plugin entry inside a marketplace
  hosted elsewhere, but npm cannot host a marketplace itself — `extraKnownMarketplaces`
  has no working `npm` marketplace source. Reach for npm only for something that's
  actually a runnable package, such as a standalone MCP server.

## Versioning

Each plugin carries its own semver in both manifests. Prefer a per-plugin `CHANGELOG.md`
(Keep a Changelog), matching bumps in both manifests, and optional annotated tags
`<plugin-name>-v<semver>` when consumers should pin.

Don't set `version` in both the plugin manifest and a marketplace entry that duplicates
it; the manifest wins silently if you do. Omitting `version` entirely makes installs
track git commits directly — fine for experiments, weaker for pinned consumers.
