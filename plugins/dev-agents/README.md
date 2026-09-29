# dev-agents

Subagents for review, testing, and debugging, plus a skill for splitting a bigger
feature across parallel workers.

## Agents

| Agent | Use when |
| --- | --- |
| [code-reviewer](agents/code-reviewer.md) | Reviewing a diff, read-only |
| [test-runner](agents/test-runner.md) | Running tests and diagnosing failures |
| [debugger](agents/debugger.md) | Tracking down and fixing a specific bug |
| [feature-builder](agents/feature-builder.md) | Building a small-to-medium feature end to end |

## Skills

| Skill | Use when |
| --- | --- |
| [feature-plan](skills/feature-plan/SKILL.md) | A feature is big enough to split across parallel subagents |
