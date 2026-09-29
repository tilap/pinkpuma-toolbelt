# Git conventions

Shared by every skill in this plugin. Read this once; each `SKILL.md` links back here
instead of repeating it.

## Commit types

[Conventional Commits](https://www.conventionalcommits.org/), format:

```
<type>(<scope>): <description>
```

- `feat` — a new capability for the end user
- `fix` — a bug fix
- `docs` — documentation only
- `refactor` — code change that neither fixes a bug nor adds a feature
- `test` — adding or correcting tests
- `chore` — tooling, dependencies, config; nothing shipped changes
- `perf` — a performance improvement
- `ci` — CI/CD configuration only

`<scope>` is optional, lowercase, one word naming the affected area (a package, a module,
a feature). Omit it when the change is repo-wide or the scope would be redundant with the
description.

`<description>` is imperative mood, lowercase, no trailing period: `add`, not `added` or
`adds`.

A breaking change appends `!` after the type/scope (`feat(api)!: ...`) and explains the
break in a `BREAKING CHANGE:` footer.

## Branch names

`<type>/<kebab-case-description>`, using the same `<type>` values as commits:
`feat/user-avatar-upload`, `fix/null-pointer-on-logout`, `chore/upgrade-eslint`.

Keep the description short (3–6 words) and specific enough that the branch name alone
tells a reader what changed, without repeating the type or restating the whole ticket
title.

## Changelog

[Keep a Changelog](https://keepachangelog.com/) format. `CHANGELOG.md` always has an
`## [Unreleased]` section that is actually used — entries land there as they ship, and
a release cuts a new dated version heading from it rather than being written from
scratch at release time.
