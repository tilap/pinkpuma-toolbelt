# Changelog

All notable changes to this plugin are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.1.0] - 2026-10-15

### Added

- `grill`, `to-spec` and `to-tickets` skills, with shared conventions for the three places
  (feature board, ticket tracker, code) and the GitHub downstream contract
  (`feature-ready`, `task-ready`, `to-resume`).
- `/feature-planning:plan`: one command that runs the three steps in order, with a
  confirmation before each one.
- A step never starts the next one: it ends by telling the person which command to type.
