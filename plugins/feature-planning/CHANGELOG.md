# Changelog

All notable changes to this plugin are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- `REVIEW.md`: three review gates (plan, ticket, feature) as agent-ready yes/no checklists.
- `to-tickets`: review sheet (rule/scenario → ticket → check) and gate A before release,
  posted as a comment on the feature; tickets gain "How to check" and "If unsure" sections
  and a test-that-fails criterion.
- `to-tickets`: tickets specify their tests so they can be written first — a "Tests" section
  with Given / When / Then cases, the interface under test, the file and framework, or an
  explicit `Tests: none` with its reason. Gate A item 12 checks it; gate B item 2 asks that the
  tests were written before the code and not edited by it.
- `grill` and `to-spec`: cross-cutting concerns (migration, security, limits, dependencies,
  compatibility) are settled and recorded.
- GitHub contract: what the planning side expects from the system's reviews.

## [0.1.0] - 2026-10-15

### Added

- `grill`, `to-spec` and `to-tickets` skills, with shared conventions for the three places
  (feature board, ticket tracker, code) and the GitHub downstream contract
  (`feature-ready`, `task-ready`, `to-resume`).
- `/feature-planning:plan`: one command that runs the three steps in order, with a
  confirmation before each one.
- A step never starts the next one: it ends by telling the person which command to type.
