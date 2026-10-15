---
description: Take a feature from a fuzzy idea to a published spec and published tickets, one confirmed step at a time (grill, then to-spec, then to-tickets).
argument-hint: "[the idea, or the link of an existing feature]"
disable-model-invocation: true
---

# Plan a feature

Take this need through the three steps of the plugin, in this conversation, one after
the other: $ARGUMENTS

If no need was given above, ask for it first. Then settle the three places (feature board,
ticket tracker, code) as the conventions say, before the first step.

Read [CONVENTIONS.md](${CLAUDE_PLUGIN_ROOT}/CONVENTIONS.md) now: it holds the rules of
every step (the three places, nothing written to disk, what may be published and how).

The skills are written to run when a person asks for them, so a step is run by reading its
file and following it exactly as written, not by calling it by name:

1. **Grill** — [skills/grill/SKILL.md](${CLAUDE_PLUGIN_ROOT}/skills/grill/SKILL.md).
   Interview until the understanding is agreed.
2. **To spec** — [skills/to-spec/SKILL.md](${CLAUDE_PLUGIN_ROOT}/skills/to-spec/SKILL.md).
   Write and publish the feature spec. Skip it when the grill concluded that the need fits a
   single ticket.
3. **To tickets** — [skills/to-tickets/SKILL.md](${CLAUDE_PLUGIN_ROOT}/skills/to-tickets/SKILL.md).
   Slice the spec, publish the tickets, then release them.

Rules of the chain:

- **One step at a time.** Finish a step, say where things stand in a few lines, and ask
  whether to go on to the next one. Do not start a step the person has not confirmed.
- **This command moves on by itself.** A skill's closing line that tells the person to type
  the next command does not apply here: once the person confirms, carry on with the next
  step by reading its file.
- **Each skill's own stops stay.** A skill that sends the person back to an earlier step
  (a decision nobody made, a blocking question) does so here too: go back, then resume.
- **Nothing is approved for the person.** Every publication needs their explicit approval of
  the exact content, and releasing the tickets to the development system is a separate
  approval. This command is a way to run the steps in order, never a reason to skip an
  approval.
- **Same conversation throughout.** The agreed understanding and the spec are carried from
  one step to the next, never fetched back or written to a file.
