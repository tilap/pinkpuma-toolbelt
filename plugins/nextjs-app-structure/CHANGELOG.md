# Changelog

All notable changes to this plugin are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- Route colocation: a component used by a single route lives in that route's
  `_components/` folder (`src/app/page123/_components/`).

## [0.1.0] - 2026-10-09

### Added

- Initial Next.js App Router layout: `src/app/`, `src/components/` with a
  presentation-only `ui/` folder, `src/config/`, and `src/env.ts` as the only reader
  of `process.env`. Pages Router and other stacks are out of scope.
