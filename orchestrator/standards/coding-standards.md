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
