# Changelog

All notable changes to the qwer orchestrator system are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/); versions are semver.

## [0.2.0] — 2026-08-25

Fixes to gaps found in real use: a role-naming fork, missing architectural standards, and — most
importantly — project-identifying detail that had leaked into the shared seed learnings despite v0.1.0
claiming they were sanitized.

### Changed
- `standards/role-rosters.md` — collapsed `unity-developer`/`unity-senior-dev` into a single canonical
  `unity-developer` name; a "senior" qualifier added no information since every role is already held to
  senior-engineer judgment, and the fork had split accumulated Tier-B learning across two files.
- `standards/coding-standards.md` — added an "Architecture" section (layering/dependency direction, no
  hidden global state, data-driven config, single-responsibility, state-the-layering-up-front) —
  previously the standard covered style only (comments, member order, naming), not design quality.
- `learnings/roles/*.md` — rewrote every file to remove project names, ticket IDs, and unreleased
  game-design/mechanic specifics that had leaked in despite the file being published studio-wide; merged
  `unity-senior-dev.md`'s content into `unity-developer.md` in fully generic form.
- `templates/subagent.template.md`, `ORCHESTRATOR.md` — made the NDA-safety rule for
  `learnings/roles/*.md` and `learnings/process-feedback.md` explicit and impossible to miss, instead of
  relying on "not project-specific trivia" being read as "no project names or design detail at all."

### Added
- `ORCHESTRATOR.md` Step 0 and `learnings/process-feedback.md` — a place for standing corrections to the
  orchestrator's *own process* (as opposed to a role's technical lessons) to persist across sessions, so
  feedback about how `/qwer` runs actually changes future runs instead of needing to be repeated.

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
