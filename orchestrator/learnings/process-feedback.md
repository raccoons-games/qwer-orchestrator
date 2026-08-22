---
purpose: standing corrections to the orchestrator playbook's own process, given by the user in past sessions
audience: Claude, at the start of every /qwer run (see ORCHESTRATOR.md Step 0)
---

# Process feedback — accumulated

Append short, timestamped, process-level entries here — what the orchestrator itself should do
differently, not a technical/domain lesson (those go in `learnings/roles/<role>.md`) and not a
project-specific fact (those stay in that project's own repo). No project names, ticket IDs, or design
specifics — this file is part of the shared, published system.

- 2026-08-25 — Role names must be decided once per stack and reused exactly, never forked into a
  seniority-qualified variant (`<role>` vs `<role>-senior-dev`) — `coding-standards.md` already requires
  senior-engineer judgment from every role, so a "senior" qualifier on the name adds no information and
  only splits accumulated Tier-B learning across two files that should be one. If Step 6 finds an
  established name for a role already in `learnings/roles/` or a target repo, use that exact name — don't
  mint a plausible-sounding alternative.
- 2026-08-25 — `coding-standards.md` needs real architectural requirements (layering, dependency
  direction, no hidden global state, data-driven config, single-responsibility), not just style rules
  (comments, member order, naming) — a "senior engineer" standard that only governs formatting isn't
  actually raising the bar on design quality.
- 2026-08-25 — Generated agents and the tech-lead skill should state something concrete about what makes
  *this* project's version of a role different (its actual architecture, constraints, established
  patterns) rather than generic boilerplate — Step 7 already says to read the repo enough to fill this in
  concretely; treat a generated file that reads as interchangeable with another project's as a sign Step
  7 wasn't actually done, not as an acceptable output.
