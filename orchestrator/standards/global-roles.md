<!--
Global (studio-wide) subagents, as opposed to the project-scoped roles in role-rosters.md. These live in
~/.claude/agents/ directly (never generated per-repo) because they're infrastructure shared across every
project, not something specific to one codebase's architecture. When Step 7 of ORCHESTRATOR.md generates
or updates a project's tech-lead skill, it should list the relevant entries from this file in that
skill's roster alongside the project-scoped roles, so every tech-lead knows these exist and when to
delegate to them instead of reaching for a project-scoped agent or guessing.
-->

# Global subagents

## `taskflow-pm`
Interfaces with the studio's TaskFlow task manager REST API (`https://taskflow.raccoons-games.us`,
`X-API-Key` header, key in `$TASKFLOW_API_KEY`). Delegate to it for: looking up a task's status/detail,
listing/searching tasks, creating a task, logging time. Holds no project-specific engineering context —
it's a thin, accurate API interface, not a planner. Any tech-lead in any repo should treat it as
available by default; it doesn't need to be generated per-project.

## Adding a new global role
Same bar as a project role (see `subagent.template.md` for tone/depth), but the file goes straight into
`~/.claude/agents/<name>.md` — not into a repo's `.claude/agents/`, and not registered in
`repos/registry.json` (that file tracks per-repo generation state; global roles have none). Document it
here so future `/qwer` runs surface it to the tech-lead they generate/update.
