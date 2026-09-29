---
name: feature-plan
description: >-
  Split a feature that spans independent areas into parallel, isolated chunks of work,
  dispatch one subagent per chunk, then synthesize and verify the result. Use for a
  feature big enough to have genuinely independent parts, on a host without a built-in
  equivalent — not for a small, single-area change (use the feature-builder agent for
  that instead).
disable-model-invocation: true
metadata:
  plugin: dev-agents
---

# Feature plan

**On Claude Code, prefer the bundled `/batch` command instead of this skill** — it does
the same worktree-isolated parallel fan-out natively, with no plugin needed. This skill
exists for hosts that don't ship an equivalent (Cursor, at the time this was written) so
the same strategy — split by ownership, agree a contract, isolate, synthesize — is
available there too, described in host-neutral prose rather than naming a specific tool.

This is "level 1" parallel feature-building. A heavier, cross-checking multi-agent
workflow — a scripted orchestrator that fans out to many more subagents and cross-checks
their results, for work too big to coordinate turn by turn — is deliberately out of scope
for this skill. Build one only once a real case demands it; don't construct that
machinery ahead of need.

## Steps

1. **Split by ownership, not by arbitrary step.** Identify chunks of the feature that
   touch disjoint sets of files or modules — a frontend piece and a backend piece, or
   several unrelated modules — not "step 1, step 2, step 3" of one sequential change.
   If the feature doesn't actually decompose this way (everything touches the same few
   files), say so and hand it to a single agent instead; forcing a split adds merge risk
   for no benefit.
2. **Write a shared contract first**, once, before dispatching anything: the interface
   between the chunks (a function signature, an API shape, a data format) that every
   chunk will build against. Chunks that each guess their own version of this contract
   is the single biggest source of wasted work in this kind of split.
3. **Isolate each chunk in its own workspace** (a separate worktree, or equivalent
   isolation the host provides) so parallel work never edits the same files.
4. **Dispatch one worker per chunk**, each given: its slice of the feature, the shared
   contract from step 2, and instructions to run its own tests before reporting back.
5. **Synthesize**: once all chunks report back, integrate them, then run the full test
   suite (not just each chunk's own tests) to catch integration issues the split
   couldn't see.
6. **Report** what each chunk did, any deviation from the shared contract that had to be
   resolved during integration, and the final test result.

## When not to use this

- The feature is small enough for one agent to hold in its head at once — just build it
  directly (or delegate to `feature-builder`).
- The "independent" parts turn out to share more state than expected once you look
  closely — merge them into fewer chunks rather than forcing a 1:1 split with the
  feature's surface description.
