<!--
Global (studio-wide) subagents, as opposed to the project-scoped roles in role-rosters.md. These run
from ~/.claude/agents/ directly (never generated per-repo, never registered in repos/registry.json)
because they're infrastructure shared across every project, not something specific to one codebase's
architecture. Where one is itself shared/reusable across the whole studio (not just this machine), its
finished agent file also ships in this repo under orchestrator/agents/<name>.md and gets seeded into
~/.claude/agents/ by INSTALL.md, same additive-only rule as learnings/roles/*.md. When Step 7 of
ORCHESTRATOR.md generates or updates a project's tech-lead skill, it should list the relevant entries
from this file in that skill's roster alongside the project-scoped roles, so every tech-lead knows these
exist and when to delegate to them instead of reaching for a project-scoped agent or guessing.
-->

# Global subagents

## `taskflow-pm`
Interfaces with the studio's TaskFlow task manager REST API (`https://taskflow.raccoons-games.us`,
`X-API-Key` header, key in `$TASKFLOW_API_KEY`). Delegate to it for: looking up a task's status/detail,
listing/searching tasks, creating a task, logging time. Holds no project-specific engineering context —
it's a thin, accurate API interface, not a planner. Any tech-lead in any repo should treat it as
available by default; it doesn't need to be generated per-project.

**The actual agent file ships in this repo at `orchestrator/agents/taskflow-pm.md`** (full REST API
reference — endpoints, request bodies, response shapes, the rich-text `description` gotcha) and gets
seeded into `~/.claude/agents/taskflow-pm.md` by `INSTALL.md`, same additive-only rule as
`learnings/roles/*.md` — a local copy that's accumulated more hard-won lessons than the seed should never
be silently overwritten by a repo update. This is the source of truth for the endpoint reference; don't
let it drift out of sync with the API docs without updating both.

## Adding a new global role
Same bar as a project role (see `subagent.template.md` for tone/depth), but the file goes straight into
`~/.claude/agents/<name>.md` — not into a repo's `.claude/agents/`, and not registered in
`repos/registry.json` (that file tracks per-repo generation state; global roles have none). Document it
here so future `/qwer` runs surface it to the tech-lead they generate/update.

**If it's studio-wide infrastructure another teammate's fresh install should also get** (as opposed to
something genuinely specific to this machine), also add the finished agent file to
`orchestrator/agents/<name>.md` in this repo and wire it into `INSTALL.md`'s seeding step — a global
role documented here but never shipped in the repo is a role every tech-lead references and no fresh
install can actually produce.
