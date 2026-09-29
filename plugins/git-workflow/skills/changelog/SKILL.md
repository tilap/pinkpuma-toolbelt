---
name: changelog
description: >-
  Update CHANGELOG.md's Unreleased section from Conventional Commits, or cut a new
  release section from it. Use when the user asks to update the changelog, or to prepare
  a release.
disable-model-invocation: true
metadata:
  plugin: git-workflow
---

# Changelog

Conventions: [../../CONVENTIONS.md](../../CONVENTIONS.md).

## Updating `## [Unreleased]`

1. If `CHANGELOG.md` does not exist, create it in Keep a Changelog format with an empty
   `## [Unreleased]` section — ask before creating it if the project might already track
   changes elsewhere (release notes, GitHub Releases only, etc.).
2. List commits since the last entry was added (`git log <last-changelog-commit>..HEAD
   --oneline`, or since the last tag if that is how the project tracks it).
3. Group by Keep a Changelog category (`Added`, `Changed`, `Fixed`, `Removed`, …), mapped
   from the commit types in `CONVENTIONS.md` (`feat` → Added/Changed, `fix` → Fixed, …).
   Skip `chore`/`ci`/`test`-only commits unless they are user-visible.
4. Write one line per entry, user-facing language — not the raw commit message. Merge
   near-duplicate entries into one.

## Cutting a release

1. Rename `## [Unreleased]` to `## [<version>] - <YYYY-MM-DD>`, then add a fresh empty
   `## [Unreleased]` above it.
2. Determine `<version>` from the project's own scheme (ask if unclear — do not assume
   semver bump rules without checking).
3. Do not create the git tag or push unless the user asks for that as a separate step.
