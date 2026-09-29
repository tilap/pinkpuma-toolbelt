# Changelog

All notable changes to this plugin are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.2.0] - 2026-09-29

### Added

- `DEFAULT_USAGE.md` so `@pinkpuma/deck` can inject project `AGENTS.md` guidance when the
  skill is available.

### Changed

- Apply documentation updates without waiting for plan approval; stop only when a project
  fact cannot be inferred.
- Point the docs checker at `../../scripts/docs_index.py` relative to the skill file.

## [0.1.0] - 2026-09-28

### Added

- Initial release: project documentation scaffold and drift check skill.
