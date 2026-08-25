# Changelog

All notable changes to the qwer orchestrator system are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/); versions are semver.

## [0.3.0] — 2026-08-25

Closes part of the gap between what v0.2.0's README/ORCHESTRATOR.md claimed ("self-learning",
"any project") and what was actually shipped, found by auditing the repo as a fresh install would
experience it. An unattended Stop-hook mechanism was drafted and briefly considered for the
self-learning gap but rejected before being enabled anywhere — no background process does this; see
"Changed" below and the README's note on what knowledge collection actually is (write-time, in-session,
not a background job).

### Changed
- `taskflow-pm` is no longer a hard dependency. `ORCHESTRATOR.md` Step 3 and the tech-lead template's
  Git workflow section now check whether `~/.claude/agents/taskflow-pm.md` actually exists and fall back
  to plain `feature/short-description` branch naming and ID-less commit messages when it doesn't, instead
  of assuming every install has the studio's task tracker configured.
- `INSTALL.md` now also seeds `learnings/process-feedback.md` (additive-only, same rule as
  `learnings/roles/*.md`) — it was added to the repo in v0.2.0 but never wired into the install steps.
- `README.md`'s file tree and install-step summary updated to match what's actually in the repo, and now
  states plainly that knowledge collection is write-time/in-session (a subagent or tech lead appending
  to `.claude/knowledge/` or `learnings/roles/*.md` itself), not an unattended background process.

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
- `prompts/learn-distill.md` — now part of the published system (previously local-only). This is the
  Stop-hook prompt that unattended-writes Tier B entries to `learnings/roles/*.md`; it was the actual
  source of the naming-fork and project-identifying-detail leaks fixed above, so every teammate's
  automated consolidation pass needs the same corrected rules, not just this machine's.

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
