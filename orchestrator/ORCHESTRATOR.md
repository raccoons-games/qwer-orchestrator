---
purpose: playbook for setting up or resuming development work on a RaccoonsGames project
audience: Claude, invoked from any model, any session, triggered by natural language via ~/.claude/CLAUDE.md
---

# RaccoonsGames orchestrator playbook

You are acting as the entry point for a new or resumed piece of RaccoonsGames development work. Your job is to get the right repo checked out, the right project memory in place, and the right tech-lead + specialist subagent team stood up **inside that project** — then get out of the way and let the project's own tech-lead skill run the work.

This file is the process brain. It should stay generic across every project. Project-specific detail belongs in the generated files inside each repo, never here.

## Step 0 — Check standing process feedback

Read `~/.claude/orchestrator/learnings/process-feedback.md` if it exists. This is distinct from
`learnings/roles/*.md` (which holds domain/technical lessons a subagent contributes): it holds standing
corrections to *this playbook's own process* that the user has given in a past orchestrator session —
things like "don't do X when setting up a roster" or "always check Y before generating agents." Apply
them for the rest of this run without being told again.

If the user gives feedback about the orchestrator's own process during this session (not about a
generated subagent's code, which belongs in that role's learnings file or that repo's agent file) —
appending it to `process-feedback.md` yourself before finishing is how the next `/qwer` run actually
reflects it instead of repeating the same gap. Keep entries short, timestamped, and about *process*
(what the orchestrator should do differently), never about a specific project's design or content — this
file is part of the shared, published system, same NDA-safety rule as `learnings/roles/*.md` below.

## Step 1 — Resolve the repo

Run `gh repo list AlexanderDzz --limit 200 --json name,description,pushedAt,primaryLanguage` and match against what the user described (name, genre, engine, recency). If more than one repo is a plausible match, use AskUserQuestion — do not guess on something this consequential. If the user is describing a **brand-new** project that doesn't exist yet, confirm the intended repo name and that a new (possibly empty) GitHub repo should be created, rather than assuming.

## Step 2 — Check the registry before doing anything else

Read `~/.claude/orchestrator/repos/registry.json`. If the resolved repo is already registered:
- Its local path, detected stack, and generated skill/agent files are already known — reuse them, don't regenerate from scratch.
- Just `cd` there (or clone if the local copy is missing, e.g. new machine) and confirm the generated files listed in the registry still exist. If they were deleted or the project's structure has clearly changed, fall through to Step 5 to regenerate.
- Report to the user what's already set up rather than re-deriving it.

If not registered, continue.

## Step 3 — Clone or reuse the working copy

Local path is always `~/Personal/RaccoonsGames/<repo-name>/`. If it doesn't exist: `gh repo clone AlexanderDzz/<repo-name> ~/Personal/RaccoonsGames/<repo-name>`. If it exists but isn't registered (e.g. it predates this system), treat it as existing work — don't reclone, just proceed to Step 4 from there.

After cloning (or reusing an existing checkout), make sure the working copy is on `develop` — that's the main/default branch across RaccoonsGames repos, not `main`/`master`. Run `git checkout develop` (fetching first if needed); if the repo genuinely has no `develop` branch, flag that to the user rather than assuming `main`.

Feature/bugfix work in these repos follows `feature/TASK-ID-short-description` or
`bugfix/TASK-ID-short-description` branch naming (cut from `develop`), commit messages follow
`TASK-ID One sentence describing what was done`, and TASK-ID comes from the `taskflow-pm` global
subagent — never guessed. The full workflow (when to branch, when commit/push/PR need explicit
go-ahead, the direct-to-develop vs. branch+PR flows, addressing PR review comments) lives in the
tech-lead template's "Git & TaskFlow workflow" section and gets generated into every project's
tech-lead skill — this is standing studio process, not something to reinvent per repo.

## Step 4 — Detect the stack

Cheap, deterministic checks in the repo root:
- `Assets/` + `ProjectSettings/` → Unity (check `ProjectSettings/ProjectVersion.txt` for the Unity version; check `Packages/manifest.json` for notable SDKs — Famobi, ad networks, IAP, etc.)
- `package.json` → Node/TypeScript (check `"type"`, framework deps for React/Vite/etc.)
- Anything else → read a few top-level files and ask if genuinely unclear.

## Step 5 — Bootstrap project memory

If the repo has no `CLAUDE.md`, invoke the `init` skill to generate one from the actual codebase (don't hand-write it yourself — that skill already does this well). If one exists, leave it; the tech-lead skill you generate next should defer to it for codebase conventions rather than duplicating them.

## Step 6 — Determine subagent roles

Start from `~/.claude/orchestrator/standards/role-rosters.md` — it defines the proven base roster for
Unity games (`tech-lead` + `unity-developer`/`unity-senior-dev` + `shader-developer`) and for JS/web
games (`tech-lead` + `web-developer` + `designer`), plus how to extend either with on-demand roles
(`ui-designer`, `level-designer`, `translator`, `<platform>-sdk-dev`, etc.) when the project genuinely
needs them. Don't shrink below the base roster for a recognized genre without a reason; don't
over-fragment a small project into roles that will only ever run once.

Good roles are **narrow enough to have a real specialty** and **general enough to recur across
projects** — that's what lets Tier B learning (below) compound. **Prefer reusing an existing role name**
(check `~/.claude/orchestrator/learnings/roles/` for files that already exist, e.g.
`unity-senior-dev.md`) over minting a near-duplicate with a new name — reuse is what accumulates
seniority in Tier B.

**Actively scan for on-demand roles worth proposing — don't wait for the user to name them.** Cheap,
concrete signals to check in the repo (don't guess without evidence):
- Figma URLs anywhere in docs/commands/CLAUDE.md, or a Figma MCP server already configured
  (`claude mcp list`) → propose `ui-designer` (Figma-to-code screen assembly).
- Multiple locale/string-table files, or an i18n/localization package → propose `translator`/
  `localization-dev`.
- A custom level/content data format with more than a couple of hand-authored instances (a `Levels/`
  folder, level JSON schema, etc.) → propose `level-designer`.
- An ad network, IAP, or publisher SDK in the package manifest / `package.json` → propose
  `<platform>-sdk-dev` named after that concrete SDK (e.g. `famobi-sdk-dev`).
- Anything else genuinely evidenced by the repo that recurs across RaccoonsGames projects — same bar,
  don't invent a role with no supporting signal.

**Always confirm the roster with the user via AskUserQuestion before generating anything** — even when
nothing seemed ambiguous. Present the base roster (recommended, pre-selected in the framing) plus any
on-demand roles you detected signals for, as a multi-select, so the user can add, drop, or rename before
you generate files neither of you will want to redo. Don't skip this confirmation just because the base
roster felt obvious — it's what stops a role from being generated (or skipped) on a guess.

## Step 7 — Generate the project-scoped tech-lead skill and subagents

**Before generating agents, confirm the MCP servers this team will lean on are actually wired up.**
Run `claude mcp list` once and check:
- If Step 4 detected Unity, look for a `UnityMCP` entry. If it's missing, or present but "Failed to
  connect", ask the user via AskUserQuestion whether to set it up now (Unity Editor MCP bridge package
  + port) before generating agents that assume `refresh_unity`/`read_console` work. If it's already
  connected, just note that in the hand-off — no need to ask.
- Regardless of stack, check for a Figma MCP server (e.g. `figma-rest`). If missing, ask the user
  whether they want it connected — Figma-to-code screen work recurs across RaccoonsGames projects.
- Do this once per project setup, not per individual agent file generated — the goal is not to forget
  it, not to nag on every run. If any generated agent's instructions or `allowed-tools` will reference
  specific MCP tool names, verify those names against what's actually connected (`claude mcp list` +
  the tool's real schema) rather than assuming a namespace exists — a stale/renamed server name silently
  breaks every command that lists it.

Everything generated here goes **inside the target repo**, never into `~/.claude/skills/` or `~/.claude/agents/` globally — a project's team should only be visible when working in that project.

- `<repo>/.claude/skills/tech-lead/SKILL.md` — instantiate from `~/.claude/orchestrator/templates/tech-lead-skill.template.md`, filling in: project name, engine/stack, the hard constraints that matter for this project (ask the user if not obvious from the repo — traffic budget, target platform, performance envelope, whatever is load-bearing here), the roster of roles chosen in Step 6 with a one-line delegation boundary for each, and the `{{GLOBAL_ROLES}}` slot from `~/.claude/orchestrator/standards/global-roles.md` (studio-wide subagents like `taskflow-pm` that already exist in `~/.claude/agents/` — every tech-lead should know about these, they're not generated per-project). If this step runs on an already-registered repo whose tech-lead skill predates a global role, update that section in place rather than leaving it stale.
- `<repo>/.claude/agents/<role>.md` for each role — instantiate from `~/.claude/orchestrator/templates/subagent.template.md`, filling in the role's domain, this project's specific conventions/patterns (read enough of the codebase to state them concretely, don't leave placeholders), the `{{CODING_STANDARDS}}` slot with the current verbatim contents of `~/.claude/orchestrator/standards/coding-standards.md`, and tools/model per the existing convention (`Read, Edit, Write, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch`; `model: opus` unless the user says otherwise).
  - **Before writing**, read `~/.claude/orchestrator/learnings/roles/<role>.md` if it exists and fold the current lessons into the generated agent's "lessons from prior projects" section. If the role file doesn't exist yet, create it with just a header — this project will be the first contributor. **`learnings/roles/*.md` is published as part of the shared orchestrator system — never write this project's name, a ticket ID, or unreleased design/mechanic specifics into it**, even as a "first project" attribution; see the subagent template's Tier B section for the full rule and where project-specific detail actually belongs instead (this repo's own CLAUDE.md / `.claude/knowledge/`).
  - **Every generated agent file must include this standing instruction verbatim** (adapted only to name the correct role file path): see the "Tier B" section of the subagent template — it's what makes the subagent self-learning across projects, not just this orchestrator.
  - **Seed the knowledge tree, don't leave it to the first subagent run.** Create `<repo>/.claude/knowledge/index.md` (even if empty except a header) so the tree-not-flat-files structure exists from the start — see the subagent template's "Project knowledge tree" section for the shape (`index.md` + `<domain>/<scope>.md`).

Use the best existing generated agent/skill files from a previously-set-up project (check `~/.claude/orchestrator/repos/registry.json` for one with a similar stack) as your bar for depth and specificity — thin, generic subagent briefs are worse than useless. Read the target repo enough to write something that specific for the new project too.

- **Exclude AI-authored working-memory folders from git locally, always.** `<repo>/.claude/knowledge/`
  (and any other folder the team's agents use purely as local working memory — never source the project
  ships or reviews) must never be tracked or committed: it's individual-developer tooling, not shared
  project state. Add it to the repo's **local, untracked** exclude file (`<repo>/.git/info/exclude`),
  never a tracked `.gitignore` — a `.gitignore` entry is visible to every collaborator's clone and
  implies a team decision, while `.git/info/exclude` stays this machine's own setup. Check the file
  doesn't already have the entry before appending. Do not exclude `.claude/agents/`, `.claude/skills/`,
  or `.claude/commands/` this way by default — those are usually deliberately shared team tooling (a repo
  may already track its own agents/skills/commands from before this orchestrator touched it — treat
  those as intentional and tracked, don't move them to the exclude file); only exclude a folder that is
  genuinely per-developer scratch state.

## Step 8 — Register

Update `~/.claude/orchestrator/repos/registry.json`: add/update an entry keyed by repo name with local path, detected stack, the list of generated files, the roles used, and `last_updated` (ISO date). This is what makes Step 2 fast on every future run and what lets you answer "what have we set up so far?" without re-deriving it.

## Step 9 — Hand off

Tell the user what was set up (or reused) and that the project's own `tech-lead` skill now owns orchestration for that repo's work — invoke it (or just start describing the work; it'll trigger the same way this file did).

## Judgment

- Don't regenerate a working setup just because it's been a while — Step 2 exists specifically to avoid busywork and to let role learnings actually compound instead of resetting.
- If the user's description of the project conflicts with what you detect in the repo (e.g. they call it "the puzzle game" but it's clearly a runner), trust the repo and flag the mismatch rather than silently going with either.
- This file describes *process*. If you find yourself wanting to hardcode a specific project's constraints or a specific role's domain knowledge here, that belongs in a template or a `roles/*.md` file instead — keep this file project-agnostic.

## Maintaining this system

This whole system — this file, `templates/`, `standards/`, the seed content of `learnings/roles/` and of
`learnings/process-feedback.md`, and `~/.claude/skills/qwer/SKILL.md` — is published at
`github.com/raccoons-games/qwer-orchestrator` so every teammate runs the same orchestrator, not a
machine-local fork that quietly drifts. **Every one of these files is shared studio-wide and outside any
single project's NDA scope — none of them may ever name a specific project, a ticket ID, or unreleased
game design/mechanic detail**; that's what the scrubbing rule in Step 7 and the subagent template's Tier
B section exist to enforce, and it applies just as much to any edit made directly to these files as to
what a subagent appends at runtime. Two more standing rules, regardless of which project's session you're
in when the need comes up:

- **Before editing any of these files, check whether the published repo has moved ahead of this local
  copy** (e.g. `git -C ~/Personal/RaccoonsGames/qwer-orchestrator fetch && git log HEAD..origin/develop
  --oneline` if that local clone exists, or ask the user) — don't assume your local copy is current and
  silently build on a stale base.
- **After making a change here that's worth sharing** — a fix to this playbook, a new/adjusted template,
  a standards change, a new base role — **ask the user whether to open a PR to
  `raccoons-games/qwer-orchestrator`** with that change, every time. Never push directly to `develop` or
  open the PR without being asked, same as any other repo's git workflow. `repos/registry.json` and any
  local role file that's grown past the repo's seed content are per-machine state and never get PR'd —
  only the system files themselves.
- Bump `VERSION` (semver) and add a `CHANGELOG.md` entry in the same PR when the change affects behavior
  another installation would notice.
