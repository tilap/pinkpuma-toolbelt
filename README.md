# pinkpuma-toolbelt

A private marketplace of [Claude Code](https://code.claude.com) and [Cursor](https://cursor.com)
plugins for everyday development work: git conventions, session handoff, project
documentation, review/test/debug subagents, and automated npm publish.

Every plugin ships as a single source shared by both tools — see
[HOWTO.md](HOWTO.md) for how that works and how to add to it.

## Plugins

| Plugin                                               | Purpose                                                                                        |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| [git-workflow](plugins/git-workflow)                 | Conventional commit messages, branch names, PR descriptions, changelog entries                 |
| [handoff](plugins/handoff)                           | Save and resume in-progress, uncommitted work across sessions                                  |
| [project-docs](plugins/project-docs)                 | Scaffold and keep a project's documentation current, sized to the project                      |
| [dev-agents](plugins/dev-agents)                     | Review, test, debug, and feature-building subagents                                            |
| [npm-publish](plugins/npm-publish)                   | Audit and set up automated npm releases (GitHub Actions + Trusted Publishing)                  |
| [spec-workflow](plugins/spec-workflow)               | Business need → agreed brief → spec issue → vertical-slice GitHub tickets                      |
| [nextjs-app-structure](plugins/nextjs-app-structure) | House layout for a Next.js App Router app: routes, components, config, and a single env module |

## Install

### Claude Code

Register this repository as a marketplace, then enable the plugins you want:

```bash
claude plugin marketplace add tilap/pinkpuma-toolbelt
claude plugin install git-workflow@pinkpuma-toolbelt
```

To enable a plugin for every project on one machine, run the commands above once; they
write to `~/.claude/settings.json`. To enable a plugin for one project only — so anyone
who clones it gets the same tools — run the install with `--scope project` instead and
commit the `.claude/settings.json` it writes:

```bash
claude plugin marketplace add tilap/pinkpuma-toolbelt --scope project
claude plugin install git-workflow@pinkpuma-toolbelt --scope project
```

This repository is private: each machine (and each collaborator) needs `git` access to
it, either an SSH key loaded in `ssh-agent` or a stored HTTPS credential (`gh auth login`
followed by `gh auth setup-git` sets one up). Claude Code uses that access directly; it
holds no credentials of its own.

### Cursor

Each plugin ships a `.cursor-plugin/plugin.json` beside its Claude Code manifest. Add a
plugin from Cursor's plugin manager by pointing it at `plugins/<name>/`.

### Pinkpuma-deck

```sh
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@git-workflow --ref git-workflow-v0.2.0
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@project-docs --ref project-docs-v0.2.0
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@handoff --ref handoff-v0.1.0
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@dev-agents --ref dev-agents-v0.1.0
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@npm-publish --ref npm-publish-v0.1.0
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@spec-workflow --ref spec-workflow-v0.1.0
npx @pinkpuma/deck add tilap/pinkpuma-toolbelt@nextjs-app-structure --ref nextjs-app-structure-v0.1.0
```


## Repository layout and contributing

See [HOWTO.md](HOWTO.md) for structure, the full **create / change / remove / release**
workflow (including developing against a local clone), and the compatibility rules that
keep every plugin working on both Claude Code and Cursor. See [AGENTS.md](AGENTS.md) for
the rules an AI agent must follow when working in this repository.
