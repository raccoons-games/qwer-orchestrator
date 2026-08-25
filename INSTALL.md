# Installing / updating qwer-orchestrator

This file is written for **Claude to execute**, not for a human to run by hand. When a user says
something like "set up the qwer orchestrator from this repo" or "update the qwer orchestrator," follow
these steps in order and report what was installed, updated, or left alone.

## 1. Get the repo locally

```
gh repo clone raccoons-games/qwer-orchestrator ~/Personal/RaccoonsGames/qwer-orchestrator
```

If `~/Personal/RaccoonsGames/qwer-orchestrator` already exists, `cd` there and `git pull` on `develop`
instead of cloning.

## 2. Install the system files (always overwrite — these are the shared source of truth)

```
mkdir -p ~/.claude/orchestrator/templates ~/.claude/orchestrator/standards ~/.claude/orchestrator/learnings/roles ~/.claude/orchestrator/repos ~/.claude/skills/qwer ~/.claude/agents

cp  <repo>/orchestrator/ORCHESTRATOR.md                    ~/.claude/orchestrator/ORCHESTRATOR.md
cp  <repo>/orchestrator/templates/*.md                     ~/.claude/orchestrator/templates/
cp  <repo>/orchestrator/standards/*.md                     ~/.claude/orchestrator/standards/
cp  <repo>/skills/qwer/SKILL.md                            ~/.claude/skills/qwer/SKILL.md
```

## 3. Seed learnings — additive only, never clobber

For each file in `<repo>/orchestrator/learnings/roles/`, copy it to
`~/.claude/orchestrator/learnings/roles/` **only if a file of that name doesn't already exist there**. A
local file that already exists has likely grown past the repo's seed content from real project work —
overwriting it would destroy accumulated Tier-B knowledge. Do the same for
`<repo>/orchestrator/learnings/process-feedback.md` → `~/.claude/orchestrator/learnings/process-feedback.md`
(same additive-only rule — it's standing process corrections, not per-repo state, but a machine that's
already accumulated its own entries shouldn't have them clobbered by the repo's seed either). Report
which files were seeded vs. skipped because they already existed.

## 3b. Seed shared global agents — additive only, never clobber

For each file in `<repo>/orchestrator/agents/` (e.g. `taskflow-pm.md`), copy it to
`~/.claude/agents/` **only if a file of that name doesn't already exist there** — same rule as learnings:
a local copy may have accumulated real-world lessons the repo's seed doesn't have yet. These are the
studio-wide subagents documented in `standards/global-roles.md`; without this step a fresh install would
have every tech-lead reference a global role (e.g. `taskflow-pm`) that doesn't actually exist yet on that
machine. Report which agents were seeded vs. skipped because they already existed.

## 4. Make sure the registry exists — never overwrite it

If `~/.claude/orchestrator/repos/registry.json` doesn't exist, create it as `{}`. If it already exists,
leave it completely alone — it's this machine's own record of which repos have already been set up.

## 5. Make sure `/qwer` is actually wired up as the trigger

Check `~/.claude/CLAUDE.md` (create it if it doesn't exist) for a section describing the orchestrator
trigger rule. If it's missing, append:

```markdown
## RaccoonsGames development orchestrator

RaccoonsGames project repos live as **private repos under the personal GitHub account `AlexanderDzz`**
(not an org). `gh` is authenticated with `repo` scope. The local clone root is
`~/Personal/RaccoonsGames/`.

The orchestrator is invoked **only** via the explicit `/qwer` command (see
`~/.claude/skills/qwer/SKILL.md`) — never by inferring intent from natural language like "let's work on
X" or "spin up a team for Y". If the user's message sounds like it wants project setup/resume work but
they didn't run `/qwer`, do the task normally and, if genuinely relevant, mention that `/qwer` exists for
that — don't self-trigger the orchestrator.
```

If the user's actual GitHub account or clone root differs from the values above (most teammates share
the same studio conventions, but confirm rather than assume if anything looks off — e.g. a different
local username), ask before writing, and adjust the snippet to match their real setup.

## 6. Record the installed version

Copy `<repo>/VERSION` to `~/.claude/orchestrator/.qwer-version` so a future update can report what
changed since this machine's last install (diff `CHANGELOG.md` between the recorded version and HEAD).

## 7. Report

Tell the user: what was freshly installed, what was updated, which learnings files were skipped because
they already existed locally (that's expected and fine, not an error), and the version now installed.
