# Changelog

All notable changes to the qwer orchestrator system are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/); versions are semver.

## [0.1.0] — 2026-08-22

Initial publish of the orchestrator system already in daily use across RaccoonsGames projects.

### Added
- `ORCHESTRATOR.md` — the full playbook: resolve repo → check registry → clone/reuse → detect stack →
  bootstrap project memory → determine subagent roles (with active on-demand-role detection and an
  always-ask AskUserQuestion confirmation before generating anything) → generate the project-scoped
  tech-lead skill and subagents → register → hand off.
- `templates/subagent.template.md`, `templates/tech-lead-skill.template.md` — the generation templates,
  including the Tier-B self-learning mechanism and the per-repo knowledge-tree convention.
- `standards/coding-standards.md`, `standards/global-roles.md`, `standards/role-rosters.md` — the
  shared, studio-wide bars every generated subagent is held to, and the proven base rosters per stack
  (Unity, JS/web), including the newly added `ui-designer` on-demand role.
- `learnings/roles/*.md` — seed Tier-B lessons for `unity-senior-dev`, `unity-developer`,
  `shader-developer`, `ui-designer`, `famobi-developer`, `backend-dev`, `frontend-dev`, `devops-dev`,
  sanitized of project-identifying detail.
- `skills/qwer/SKILL.md` — the `/qwer` trigger command.
- `INSTALL.md` — install/update procedure for Claude to execute directly.
- "Maintaining this system" section in `ORCHESTRATOR.md` — the standing rule that any change to this
  system asks before opening a PR here, and checks for upstream drift before assuming the local copy is
  current.
