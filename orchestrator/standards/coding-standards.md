<!--
Shared coding-standards block. The orchestrator pastes this verbatim into every generated
`<repo>/.claude/agents/<role>.md` (subagent.template.md's {{CODING_STANDARDS}} slot) and it also backs
the tech-lead's review checklist. Edit here once, not per-repo. Stack-specific subsections apply only
when that stack is in play — a JS/web subagent ignores the Unity subsection entirely.
-->

# Coding standards — all RaccoonsGames subagents

You are a **senior engineer**, not a mirror of whatever quality already exists in the repo. If the
existing codebase has sloppy patterns (comment spam, dead code, inconsistent member order, mixed
languages), match its *architecture and idiom* where the task doesn't touch that code, but never let it
lower your own bar in the code you write or touch. Flag systemic sloppiness to the tech lead instead of
silently propagating it.

## Architecture
- **Separate pure logic from engine/presentation.** Simulation, rules, and state transitions belong in
  plain classes with no engine references (no `MonoBehaviour`, no `UnityEngine.*`, no DOM/React state)
  wherever the codebase's existing structure allows it — this is what makes logic unit-testable and
  headlessly verifiable. Views/presenters read from and drive that layer; they do not hold gameplay
  rules themselves. If the existing codebase doesn't separate these, don't block on introducing the
  split mid-task, but don't add new rules to the engine-coupled layer either — flag the gap to the tech
  lead instead of compounding it.
- **Dependency direction is one-way and inward.** Presentation depends on logic/domain, never the
  reverse; low-level utilities never depend on feature code; a shared/core layer never imports from a
  feature-specific one. If a change would require the domain layer to reference a view type or a
  feature module to reference a sibling feature module, that's a design smell — route it through an
  interface/event the dependency direction already allows, or flag it rather than adding the backwards
  reference.
- **No hidden global mutable state.** Prefer passing dependencies explicitly (constructor/DI) over
  static singletons, service locators, or ambient globals — see the DI note below for the Unity-specific
  form of this rule. A static is acceptable only for genuinely stateless utilities or true constants.
- **Data-driven over hardcoded.** Tunable values that a designer or a different build config would
  plausibly want to change (balance numbers, timings, feature flags, per-platform limits) belong in
  data/config, not literals buried in logic — unless the codebase has no such config layer at all, in
  which case match its existing convention rather than inventing one mid-task.
- **A class has one reason to change.** Split the moment a class is doing two unrelated jobs (e.g.
  "computes the result" and "renders the result", or "owns network I/O" and "owns business rules").
  Prefer several small, named collaborators over one class that grows a new responsibility per feature.
- **State this project's actual layering up front, don't assume it.** When a project's existing
  architecture isn't self-evident from a quick read, the tech lead states it explicitly (in the
  generated tech-lead skill / subagent's "How this codebase is built" section) before work starts —
  guessing at boundaries mid-task is how a change ends up straddling a layer it shouldn't.

## Comments
- No comment spam. Do not narrate what the code does — intention-revealing names and small methods
  already say that.
- Comment only the genuinely non-obvious "why": a hidden constraint, a workaround for a specific
  external bug, a temporary hack with a reason it's temporary, an invariant that would surprise a
  reader. If removing the comment wouldn't confuse the next reader, don't write it.
- Never leave "removed X" / "TODO from old code" archaeology comments.

## Language
- All code, identifiers, comments, commit messages, and docs are English only — no exceptions, even if
  surrounding project docs (GDD, specs) are in another language. Translate concepts, don't transliterate.

## Member ordering (C#, and any language with an equivalent concept)
Always, top to bottom within a type:
1. Constants
2. Static fields
3. Private (instance) fields
4. Public properties
5. Static methods
6. Public methods
7. Private methods

Constructors go with public methods (immediately after properties, before other public methods) unless
the codebase's existing convention places them elsewhere consistently — match existing convention only
when it's already consistent; if the file you're editing is already ordered this way, keep it that way;
if it's inconsistent, order new code correctly without a drive-by reformat of unrelated members.

## Unity-specific
- Prefer dependency injection (constructor/[Inject]) over singletons, static service locators, or
  `FindObjectOfType`/`GameObject.Find`. If the project has an established DI convention (Zenject,
  hand-rolled `[Inject]`, etc.), use it — don't introduce a second DI mechanism.
- Prefer async idioms (UniTask if the project has it, else `Task`/coroutines matching existing
  convention) over blocking waits, manual polling loops, or callback pyramids — unless the codebase's
  established pattern is coroutines end-to-end, in which case match that instead of introducing a new
  async style mid-codebase.
- Self-documenting, clean code: small methods, early returns, no God-classes. Split a class the moment
  it's doing two unrelated jobs.

## Self-documenting, clean code (all stacks)
- Intention-revealing names over comments.
- Small functions/methods with a single responsibility; early returns over nested conditionals.
- No dead code, no commented-out code, no speculative abstractions for hypothetical future needs.
