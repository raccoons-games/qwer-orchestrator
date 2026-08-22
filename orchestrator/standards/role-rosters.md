<!--
Base subagent rosters per stack. Step 6 of ORCHESTRATOR.md starts from these instead of inventing a
split from scratch each time — they're the proven starting point; extend per-project when the user
names an extra scope (level-designer, translator, SDK integration, etc.), don't shrink below them
without a reason. Keep this list itself generic — a role's project-specific depth belongs in the
generated agent file and in learnings/roles/<role>.md, never here.
-->

# Base subagent rosters

## Unity games
Always start with:
- **`tech-lead`** (skill, not agent) — orchestrates, decomposes, delegates, reviews, integrates.
- **`unity-developer`** (or `unity-senior-dev` if already the established name in
  `learnings/roles/` — prefer reuse over renaming) — gameplay, engine, DI wiring, views/UI,
  presenters, performance. The default owner of anything not carved out below.
- **`shader-developer`** — hand-authored ShaderLab/HLSL shaders and the C#/MaterialPropertyBlock glue
  that drives them at runtime (VFX, sprite/unlit effects, hue-shift, noise, scrolling, distortion).
  Explicitly NOT gameplay logic, DI wiring, or Core/simulation work.

Add on demand, named after the concrete scope (don't pre-create speculatively):
- **`ui-designer`** — Figma-to-Unity screen assembly (structure, sprite export/import, anchor-driven
  resolution-independent RectTransform layout, Play-mode verification against the design), once the
  project has a Figma-sourced UI pipeline worth a dedicated role. Explicitly NOT gameplay logic or
  DI/Zenject wiring beyond a screen's own bindings.
- **`level-designer`** — level data/content authoring and balancing, once the project has a level
  format worth a dedicated role (not just a couple of hand-edited files).
- **`translator`** / **`localization-dev`** — localization pipeline and string/asset management, once
  the project ships in more than one language.
- **`<platform>-sdk-dev`** (e.g. `famobi-sdk-dev`, `ironsource-sdk-dev`) — named after the platform, one
  per third-party SDK actually integrated.

## JavaScript / web games
Always start with:
- **`tech-lead`** (skill, not agent) — same role as above.
- **`web-developer`** — gameplay/engine code, build tooling, state management, the game loop itself.
- **`designer`** — UI/UX and visual layer: screens, layout, art integration, animation/transition
  polish, responsive behavior. Explicitly NOT game-loop or state-management logic.

Add on demand, same pattern as Unity: `level-designer`, `translator`, `backend-dev` (if the game has a
server component beyond static hosting), platform-SDK roles named after the platform.

## Judgment
- Reuse an existing role name over minting a near-duplicate — check `learnings/roles/` first.
- Don't fragment a small project into roles that will only ever run once; two generalist roles beat
  four one-shot specialists.
- These rosters are a floor for the named genres, not a ceiling — add scopes the project genuinely
  needs, but name them after the concrete domain, not a vague catch-all.
