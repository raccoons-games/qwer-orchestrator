---
purpose: cross-project accumulated lessons for the unity-senior-dev role
audience: Claude, any RaccoonsGames project generating or running a unity-senior-dev subagent
---

# unity-senior-dev — accumulated lessons

Contribute short, timestamped, sourced entries here as you learn things worth carrying forward to the next project (not project-specific trivia — that belongs in the repo's own CLAUDE.md).

## 2026-08-18 — a Unity project with an engine-pure simulation core — run engine-pure asmdefs headless with `dotnet test`

If a project keeps its simulation/logic in an asmdef with `noEngineReferences` and its EditMode tests
only reference NUnit + that asmdef, you don't need the Unity Editor (or a live MCP connection) to
compile-check and run the whole suite. Generate a throwaway SDK-style csproj in the scratchpad with
`EnableDefaultCompileItems=false` and `<Compile Include="…/Scripts/Core/**/*.cs" />` +
`…/Tests/EditMode/**/*.cs`, `PackageReference` NUnit + NUnit3TestAdapter + Microsoft.NET.Test.Sdk,
then `dotnet test`. Seconds per run, no Editor domain reload.

Two multipliers on this:
- **Always run the same suite against `git archive HEAD` in a second scratch project first.** Pre-existing
  red tests are common; a side-by-side baseline turns "did I break it?" into a one-command answer.
- **For deterministic sims, compare the failing assertion's *actual float value*, not just pass/fail.**
  A bit-identical actual before and after your change is much stronger evidence you didn't perturb the
  RNG/force pipeline than a green suite is.

Also usable for ad-hoc probing of `private` geometry helpers: a tiny console project over the same
globs + `RuntimeHelpers.GetUninitializedObject` + reflection lets you table-test a new collision/
occlusion predicate in isolation (remember to hand-set fields whose initializers get skipped).

## 2026-08-18 — a Unity project with an engine-pure simulation core — gate *all* sibling long-range forces at once

When fixing "attraction works through walls", grep for every long-range force in the same family
before editing — this sim had three (same-color gravity, merge magnet, booster magnet) sharing one
conceptual rule. Fixing only the one the bug report names produces a new, subtler bug (a wall that
blocks one pull but not the other reads as broken geometry). Compute the occlusion predicate once per
pair and lazily (`(forceA || forceB) && Blocked(...)`) so an O(n²) loop doesn't pay for geometry on
pairs already rejected by distance/color. Do NOT extend the gate to the contact/merge branch —
touching next to a wall must still merge.

## 2026-08-19 — a Unity project with an engine-pure simulation core — compile-check *engine-referencing* code without the Editor

The prior entry covers headless testing of `noEngineReferences` asmdefs. The Editor-dependent half is
also solvable: Unity leaves generated SDK-style `.csproj` files at the repo root (`SandShoot.Game.csproj`
etc., gitignored) that already reference the correct `UnityEngine.*`/package DLLs from `Library/`.
`dotnet build <Asmdef>.csproj` compiles the real Game layer in ~2s with no Editor running. Caveat: the
csproj is a *stale snapshot* — a file you just created isn't in it, and you get a bogus CS0246 for your
own new type. Add a `<Compile Include="…" />` line for the new file (Unity regenerates the csproj
anyway) and rebuild. Afterwards delete the `obj/` and `Temp/bin` output the build drops.

This turns "MCP bridge is down, I can't verify" into "compilation verified, only Play-mode behaviour
is unverified" — a much better honest report. Check `lsof -nP -iTCP:<mcp-port> -sTCP:LISTEN` first to
know which situation you're in instead of discovering it mid-task.

## 2026-08-19 — a Unity project with an engine-pure simulation core — lazy runtime host beats a prefab edit when the Editor is down

Needed a new UI particle effect hosted on a HUD screen prefab, with no Editor to author the child
object. Instead of hand-editing prefab YAML (fragile GUIDs/fileIDs), keep the serialized field for the
designer and lazily create the host object on first use (`field != null ? field : Create()`), which
these codebases already do widely for authored-else-runtime views. Zero prefab/scene diff, no YAML
risk, and it stays inspector-authorable later. Remember the runtime-created component gets **no
Zenject injection** — mark its dependencies `[Inject(Optional = true)]` with a sane fallback, or the
effect silently no-ops in a build.

## 2026-08-19 — a Unity project with an engine-pure simulation core — multi-phase FX curves belong in the pure sim as one function

A designer-facing "pulse 3× then a wave sweeps it away" celebration is tempting to implement as view
state (a coroutine per phase, flags, a second timer). Don't. Keep the sim's single elapsed-time field
and expose the whole timeline as one pure `Blend(cellPosition, elapsed)` on the engine-pure layer:
the view stays a stateless `Lerp(color, white, blend)` line, the curve is unit-testable/plottable in a
2-minute headless console harness, and presenters only need one exposed `TotalDuration` constant to
`WaitForSecondsRealtime` past the effect instead of polling sim state or wiring new events.
Two things that harness catches instantly and eyeballing does not: (1) phase-boundary discontinuity —
the last pulse must *settle* at exactly the value the next phase starts from, or the picture visibly
snaps; (2) a sweeping front parameterised `-band → 1` leaves the far cell jumping to 0 at the end —
it must travel `-band → 1+band` so the trailing edge clears the last cell within the duration.

## 2026-08-19 — a Unity project with an engine-pure simulation core — deleting an inspector-wired view feature: finish the job in the prefab

When a ticket says "remove effect X", removing the C# is only half of it. Two failure modes that both
ship silently green:
1. The authored GameObject that hosted it stays in the prefab/scene. If its *only* hide path was the
   `Reset*()` method you just deleted (these codebases hide FX by `SetActive(false)` on level rebind,
   not by an idle state), it now stays visible forever with stale text. Set `m_IsActive: 0` on that
   GameObject and strip its `fileID` refs out of the serialized component block — both are safe,
   surgical one-line YAML edits, unlike deleting a subtree (fragile fileID/child-list surgery).
2. The Core-side event that fed it. Leave the event (cheap hook, no consumers) but make sure the
   view's `Bind`/`Unsub` pair loses *both* the `+=` and the `-=` — dropping only one is an easy diff
   slip and leaks the view on every level rebind.

Also: package effects like Easy Text Effects bind assets to code by a serialized **`effectTag` string
field**, not by asset name. To verify "is the idle animation actually configured?" without the Editor,
resolve the GUIDs in the component's `globalEffects` list via `grep -rl "guid: <g>" Assets | grep .meta`
and read the asset's `effectTag` + parameters directly. Faster and more reliable than opening the
Inspector.

## 2026-08-19 — a Unity project with an engine-pure simulation core — author ScriptableObject FX assets by generating YAML, not by hand

A "give this popup 8 animation variants" ticket needed ~45 Easy-Text-Effects `.asset` files. Hand-editing
that much YAML is error-prone and Editor-free anyway; a ~150-line Python generator (one `add(name, kind,
tag, duration, stagger, curveKeys, tail)` call per asset, `uuid4().hex` for the `.meta` GUID, and a
printed manifest of name→GUID) produced all of them plus the prefab list patch in one deterministic pass,
and re-tuning a curve is a one-line edit + rerun. Two things that make it safe:
- **Renaming an existing asset keeps its GUID**, so `git mv` + editing `m_Name`/the tag field re-purposes
  a legacy asset into a new naming scheme with zero prefab/scene churn. Prefer that over delete+recreate.
- Serialized *lists* on a component (e.g. `globalEffects`) are plain YAML sequences — insert entries
  right before the next scalar key of the same block; locate the block by index, not by regex, when the
  same key name appears on several components in the file.

Generic rule for tween/text-effect packages with a "OneTime" mode: the effect's contribution vanishes the
frame it completes, so every *entrance* animation must end at identity (scale 1, offset 0, angle 0) or the
element visibly snaps. Exits are exempt only if the code hides the object before the effect completes.

## 2026-08-19 — a Unity project with an engine-pure simulation core — DOTween in asmdef projects: the Modules half is invisible

Dropping the Asset Store DOTween into `Assets/Plugins/Demigiant/` gives asmdef assemblies **less API than
you expect**. `DOTween.dll` is auto-referenced (no `.asmdef` reference entry needed, and no
`DOTween.Init()`/installer wiring — confirm with a headless `dotnet build <Asmdef>.csproj`, not by
assuming), but `Modules/*.cs` (UI, TMP, Sprite, Physics shortcuts) live under `Assets/Plugins` with no
asmdef, so they compile into `Assembly-CSharp-firstpass` — which an asmdef assembly **cannot** reference.
Net effect: `Transform` shortcuts (`DOScale`/`DOPunch*`/`DOLocalRotate`) work, but `TMP_Text.DOFade`,
`Image.DOColor`, `RectTransform.DOAnchorPos` do not. Reach for the generic
`DOTween.To(() => x.color, c => x.color = c, target, dur)` instead of adding an asmdef for the plugin.

Game-feel notes worth reusing when a designer says "the popup is too calm":
- The escalation the player actually reads is a **scale pop + `DOPunchPosition` + `DOPunchRotation`
  joined at t=0**. A scale-only tween never reads as impact no matter how aggressive its curve.
- Tune the *visible peak*, not the tween target: `Ease.OutBack` overshoots ~1.1× past its target and
  `Ease.OutElastic` ~1.15×, so a "1.8 overshoot" spec with elastic actually lands ~2.1× and can shove
  a world-space label off screen. Author the target below the requested peak and say so.
- "Different every time" is cheaper and stronger as **continuous `Random.Range` inside a per-tier
  parameter band** than as a pool of authored presets — and it deletes the whole asset-authoring layer.
- Replacing a per-frame timer state machine with one owned `Sequence` means: cache the authored
  `localScale` in `Awake` (a killed sequence leaves the transform anywhere), `Kill()` the field before
  rebuilding / in `Reset*()` / in `OnDestroy()`, and remember `Kill()` defaults to `complete: false` so
  `OnComplete` won't fire and undo the state you just set. Sequences may not contain infinite loops —
  a "hold with idle wobble" is a finite yoyo whose period is `hold / loops`.

## 2026-08-19 — a Unity project with an engine-pure simulation core: verify third-party DLL semantics from IL, don't guess
When a ticket hinges on "does this closed-source plugin API misbehave with these args?", disassemble
instead of reasoning from docs: `MONO_PATH=<UnityEditor>/PlaybackEngines/AndroidPlayer/Variations/mono/Managed
monodis --output=x.il <Plugin>.dll`. Without `MONO_PATH` monodis segfaults resolving `UnityEngine`, and
the Editor's own `Unity.app/Contents/Managed` no longer holds `UnityEngine.dll` in Unity 6 — the
PlaybackEngines variation does. Concrete result: DOTween's `SetEase(Ease, float amplitude, float period)`
only stores three fields (Flash is the lone special case) and `EaseManager.Evaluate` dispatches through
`switch(ease - 1)` where non-Back/Elastic branches never read those args — so passing `0f, 0f` with
`Ease.OutQuad` is provably inert. Took ~2 minutes and turned a "should be fine" into a fact.

Corollary for validation when the Editor is open: the Unity lockfile blocks a second `-batchmode`
instance, but a pure-C# `noEngineReferences` Core asmdef + its NUnit EditMode tests run under plain
`dotnet test` from a throwaway SDK-style csproj that globs the two source trees. Keep a known-failing
test's *exact actual value* as a determinism canary — bit-identical across a change is strong evidence
the sim wasn't perturbed.

## 2026-08-19 — a Unity project with an engine-pure simulation core: when the Editor is unreachable, clone an authored object instead of writing prefab YAML
Hand-writing a brand-new world-space `TextMeshPro` GameObject into a `.prefab` blind (MeshRenderer +
material + font asset GUIDs + sorting) is the kind of change whose mistakes only surface when someone
opens the Editor. `Instantiate(existingLabel, transform)` in `Awake`, then overriding just the few
properties that differ (fontSize, wrapping, colour alpha, z), gives a second fully-configured object
with zero new authoring surface and zero missing-reference risk — and it stays correct if a designer
later retargets the original's font. Cheap enough that it is often the right call even with the Editor
open. Two things to get right: clone in `Awake` (before any tween has moved/tinted the source) and
reset `localScale`/`localRotation` from the cached authored values.
Related sizing lesson: for a full-screen "ticker" that must start and end fully off-screen, do not use
fixed viewport offsets like `1.15f` — the element is often wider than the screen. Take the viewport 0/1
world edges and pad by `text.GetPreferredValues().x * 0.5f * lossyScale.x`, so it is independent of
string length, font scale and aspect ratio.

## 2026-08-20 — a Unity project with an engine-pure simulation core — verify a merge resolution by diffing against *both* parents

"No conflict markers left + it compiles" is a weak check on a hand-resolved conflict: the real failure
mode is silently dropping one side's hunk. Two cheap, decisive checks:
- `git diff <parentA> <mergeCommit> -- <paths>` and `git diff <parentB> <mergeCommit> -- <paths>`. Each
  diff should contain *only* the other parent's intended changes. An empty diff against the upstream
  parent for files you didn't touch is proof upstream survived verbatim.
- For a file where both sides added scattered call sites (e.g. an audio/analytics service sprinkled
  through a view), compare **call-site inventories**, not the diff:
  `git show <parent>:<file> | grep -o "_sfx?\.[A-Za-z]*" | sort | uniq -c` vs the same on the merged
  file. Equal multisets = nothing dropped, in one glance.

Two traps hit while doing this:
- **Unity's `Editor.log` is append-only and full of stale `error CS…`.** Don't read the last matching
  error as current state — check the *line number* of the last error against `wc -l`; thousands of
  clean lines after it means it's from an old edit session. `dotnet build` on the generated csprojs is
  the authoritative answer.
- **Conflict-marker greps must force text mode and cover untracked files.** `git grep` has no
  `--binary-files=text` (it's `-a`), plain `grep -I` *skips* the Unity YAML/asset files you most want
  to check, and zsh aborts a whole command line on an unmatched glob like `*.json`. Use
  `find … -prune -o -type f -print0 | xargs -0 grep -laE '^(<<<<<<<|>>>>>>>)'` and a separate pass for
  `^=======$`, pruning `Library/ Temp/ Logs/ obj/ .git/`.

Also: a merge that brings in a new `Assets/<Folder>/` can arrive **missing the folder's own `.meta`**
(upstream committed the contents but not `Assets/Audio.meta`). It shows up as a lone `??` in
`git status` and is easy to dismiss as editor noise — commit it, or every machine generates a
different folder GUID.
