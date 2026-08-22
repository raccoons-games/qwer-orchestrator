# qwer-orchestrator

The RaccoonsGames `/qwer` development orchestrator — the process Claude follows to clone/resume a
project, stand up its project-scoped tech-lead + specialist subagent team, seed its knowledge tree, and
keep all of that in sync across every repo and every machine on the team.

This repo *is* the source of truth for that system. `~/.claude/orchestrator/` and
`~/.claude/skills/qwer/` on any teammate's machine are a local installation of what's in here — not a
fork, not a personal copy to diverge from silently.

## What's in here

```
orchestrator/
  ORCHESTRATOR.md            the playbook itself — read this to understand the whole flow
  templates/                 subagent.template.md, tech-lead-skill.template.md
  standards/                 coding-standards.md, global-roles.md, role-rosters.md
  learnings/roles/           seed Tier-B lessons per role (sanitized of project-identifying detail)
skills/qwer/SKILL.md          the /qwer slash command that triggers the whole thing
```

Not in here, and never will be: `repos/registry.json` (per-machine state — which repos you've already
set up, their local paths) and the *live*, ever-growing `learnings/roles/*.md` on your machine once
they've accumulated more than this repo's seed content. Those are local/personal-machine state, not
shared system.

## Install — for a teammate setting this up for the first time

Tell your Claude:

> Set up the qwer orchestrator from https://github.com/raccoons-games/qwer-orchestrator

Claude should follow `INSTALL.md` in this repo, which:
1. Clones/pulls this repo to `~/Personal/RaccoonsGames/qwer-orchestrator`.
2. Copies `orchestrator/ORCHESTRATOR.md`, `orchestrator/templates/`, `orchestrator/standards/` into
   `~/.claude/orchestrator/` (always overwritten — these are the system files).
3. Seeds `orchestrator/learnings/roles/*.md` into `~/.claude/orchestrator/learnings/roles/` — only the
   files that don't already exist locally; never clobbers an existing, grown learnings file.
4. Makes sure `~/.claude/orchestrator/repos/registry.json` exists (creates `{}` if missing) — never
   touches it if present.
5. Copies `skills/qwer/SKILL.md` into `~/.claude/skills/qwer/SKILL.md` (always overwritten).
6. Adds the orchestrator-trigger paragraph to your `~/.claude/CLAUDE.md` if it isn't there yet, so
   `/qwer` is actually recognized as the entry point and nothing else self-triggers it.
7. Records the installed version so future updates can tell if you're behind.

## Update — for a teammate who already has it installed

Tell your Claude:

> Update the qwer orchestrator from the repo

Same steps as install, run again — it's idempotent. Your accumulated `learnings/roles/*.md` and
`repos/registry.json` are untouched either way.

## Versioning

Semver in `VERSION`, tagged on the `develop` branch (this repo doesn't use `main` — RaccoonsGames repos
default to `develop`). See `CHANGELOG.md` for what changed release to release. Bump the version any time
`ORCHESTRATOR.md`, a template, or a standards file changes in a way another installation should pick up.

## Contributing a change

If your Claude proposes an improvement to the orchestrator itself while you're using it in some other
project — a fix to `ORCHESTRATOR.md`, a new base role, a standards tweak — it should ask you before
opening a PR here, and should check whether this repo has moved ahead of your local install before
assuming its own copy is current. See the "Maintaining this system" section in `ORCHESTRATOR.md`.
