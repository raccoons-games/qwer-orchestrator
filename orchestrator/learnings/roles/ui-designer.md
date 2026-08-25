---
purpose: cross-project accumulated lessons for the ui-designer role
audience: Claude, any RaccoonsGames project generating or running a ui-designer subagent
---

# ui-designer — accumulated lessons

Contribute short, timestamped, generic entries here as you learn things worth carrying forward to the
next project — no project names, ticket IDs, or unreleased design specifics (see the subagent
template's Tier B section for why); project-specific trivia belongs in the repo's own CLAUDE.md /
`.claude/knowledge/`.

- Map Figma `constraints` to RectTransform anchors instead of baking one canvas-space `(x, y, w, h)` per
  node from a single Figma frame size and a fixed reference resolution — the latter reproduces the
  design at exactly one aspect ratio and drifts (clips/overlaps/detaches) everywhere else. Figma's
  `constraints` field (`horizontal`: `LEFT`/`RIGHT`/`LEFT_RIGHT`/`CENTER`/`SCALE`; `vertical`:
  `TOP`/`BOTTOM`/`TOP_BOTTOM`/`CENTER`/`SCALE`) is the designer's actual intent for how a node behaves as
  its parent resizes, and maps directly onto anchors: `LEFT`/`TOP`/`RIGHT`/`BOTTOM` → anchor pinned to
  that edge, fixed offset, fixed size; `CENTER` → anchors at 0.5 on that axis, offset from center, fixed
  size; `LEFT_RIGHT`/`TOP_BOTTOM` (stretch) → anchors span 0..1 on that axis with margins, size follows
  the parent; `SCALE` → anchors proportional to the node's own position/size within its parent — the one
  case where baking in a resolution-independent fraction is actually correct. Compute anchors relative to
  each node's *actual Figma parent frame*, not the outer screen frame, so nesting composes correctly.
  Only fall back to a fixed center-anchor + pixel offset when a node's constraints are missing or
  ambiguous, and say so explicitly rather than silently degrading every node to that. After assembly,
  verify at more than one aspect ratio (resize the game view / drive the canvas scaler) — overlay UI
  often doesn't render in an edit-mode camera capture, so check it in Play mode.

- When exporting Figma assets via an image-export API/tool, prefer its native transparent-PNG export over
  a flood-fill/chroma-key workaround — most such tools export transparency by default, and a manual
  keying step is only needed if a node genuinely bakes an opaque parent fill into the export.

- Keep dynamic content dynamic: a live view backed by a runtime-rendered texture, or any designer-baked
  number/price that actually changes at runtime, must be wired to its real data source when the screen
  opens — never baked as a static sprite/label just because that's what the Figma export looks like.

- Prefer transparent, raycastable hit targets sized from the Figma node's own bounds over baking
  interactive regions into background art — keeps hit areas correct if the art changes and keeps the
  Figma bounds as the single source of truth for both visuals and interaction.
