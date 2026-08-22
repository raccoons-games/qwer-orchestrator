---
purpose: cross-project accumulated lessons for the unity-developer role
audience: Claude, any RaccoonsGames project generating or running a unity-developer subagent
---

# unity-developer — accumulated lessons

`unity-developer` is the single canonical name for this role — see `standards/role-rosters.md`. Contribute
short, timestamped, generic entries here as you learn things worth carrying forward to the next
project — no project names, ticket IDs, or unreleased game-design/mechanic specifics (see the subagent
template's Tier B section for why); project-specific trivia belongs in the repo's own CLAUDE.md.

- For Unity WebGL, use `!Input.touchSupported` to detect desktop browser vs mobile browser at runtime.
  `Application.isMobilePlatform` and `SystemInfo.deviceType` are unreliable in WebGL builds and should
  not be used for this purpose. Evaluate once in `Awake`/`Start`, store in a static property (e.g.
  `PlatformUtils.IsDesktopWeb`), and keep it a one-time decision rather than a per-frame check.

- Run engine-pure asmdefs headless with `dotnet test`: if a project keeps its simulation/logic in an
  asmdef with `noEngineReferences` and its EditMode tests only reference NUnit + that asmdef, you don't
  need the Unity Editor (or a live MCP connection) to compile-check and run the whole suite. Generate a
  throwaway SDK-style csproj in the scratchpad with `EnableDefaultCompileItems=false` and
  `<Compile Include="…/Scripts/Core/**/*.cs" />` + `…/Tests/EditMode/**/*.cs`, `PackageReference` NUnit +
  NUnit3TestAdapter + Microsoft.NET.Test.Sdk, then `dotnet test`. Seconds per run, no Editor domain
  reload. Two multipliers on this: always run the same suite against `git archive HEAD` in a second
  scratch project first (a side-by-side baseline turns "did I break it?" into a one-command answer); for
  deterministic sims, compare the failing assertion's *actual float value*, not just pass/fail — a
  bit-identical actual before and after your change is much stronger evidence you didn't perturb an
  RNG/force pipeline than a green suite is. The same technique is usable for ad-hoc probing of `private`
  geometry/logic helpers: a tiny console project over the same globs + `RuntimeHelpers.GetUninitializedObject`
  and reflection lets you table-test a new predicate in isolation (remember to hand-set fields whose
  initializers get skipped).

- Compile-check *engine-referencing* code without the Editor: Unity leaves generated SDK-style `.csproj`
  files at the repo root (gitignored) that already reference the correct `UnityEngine.*`/package DLLs
  from `Library/`. `dotnet build <Asmdef>.csproj` compiles the real engine-coupled layer in ~2s with no
  Editor running. Caveat: the csproj is a *stale snapshot* — a file you just created isn't in it, and
  you get a bogus CS0246 for your own new type. Add a `<Compile Include="…" />` line for the new file
  (Unity regenerates the csproj anyway) and rebuild. Afterwards delete the `obj/`/`Temp/bin` output the
  build drops. This turns "MCP bridge is down, I can't verify" into "compilation verified, only
  Play-mode behaviour is unverified" — check `lsof -nP -iTCP:<mcp-port> -sTCP:LISTEN` first to know which
  situation you're in instead of discovering it mid-task.

- When several similarly-shaped long-range/area forces or checks share one conceptual rule (e.g.
  multiple attraction or magnet-style effects that should all respect the same occlusion/line-of-sight
  rule), grep for every sibling in the family before fixing a bug report about just one of them — fixing
  only the named instance produces a new, subtler bug elsewhere in the same family. Compute a shared
  predicate once per pair and lazily (`(conditionA || conditionB) && Blocked(...)`) so an O(n²) loop
  doesn't pay for the expensive check on pairs already rejected by a cheaper filter (distance, category,
  etc.). Don't over-extend the gate to an adjacent branch (e.g. a contact/merge case) that has different
  correct behavior near the same obstacle.

- A lazy runtime host beats a prefab edit when the Editor is down: to host a new runtime-created
  child/effect object on an existing prefab with no Editor available to author it by hand, keep the
  serialized field for the designer and lazily create the host object on first use
  (`field != null ? field : Create()`) rather than hand-editing prefab YAML (fragile GUIDs/fileIDs).
  Zero prefab/scene diff, no YAML risk, stays inspector-authorable later. Remember the runtime-created
  component gets **no DI/Zenject injection** — mark its dependencies `[Inject(Optional = true)]` with a
  sane fallback, or the effect silently no-ops in a build.

- Multi-phase FX/animation curves (e.g. several sequential visual beats forming one timeline) belong in
  the pure sim/logic layer as one function, not as view state (a coroutine per phase, flags, a second
  timer). Keep a single elapsed-time field and expose the whole timeline as one pure
  `Blend(input, elapsed)` on the engine-pure layer: the view stays a stateless interpolation line, the
  curve is unit-testable/plottable in a headless harness, and presenters only need one exposed
  `TotalDuration` constant to wait past the effect instead of polling sim state or wiring new events.
  Two things a harness catches instantly that eyeballing does not: (1) phase-boundary discontinuity — an
  earlier phase must *settle* at exactly the value the next phase starts from, or the picture visibly
  snaps; (2) a sweeping/traveling front parameterized `-band → 1` leaves the far end jumping at the end —
  it must travel `-band → 1+band` so the trailing edge clears cleanly within the duration.

- Deleting an inspector-wired view feature: finish the job in the prefab, not just the C#. Two failure
  modes that both ship silently green: (1) the authored GameObject that hosted it stays in the
  prefab/scene — if its only hide path was a method you just deleted, it now stays visible forever with
  stale content; setting the relevant flag inactive and stripping its `fileID` refs from the serialized
  component block are safe, surgical one-line YAML edits, unlike deleting a subtree (fragile
  fileID/child-list surgery). (2) The event that fed it — leave the event (cheap hook, no consumers) but
  make sure the view's subscribe/unsubscribe pair loses *both* sides, not just one; dropping only one is
  an easy diff slip and leaks the view on every rebind. Also: tween/text-effect asset packages often bind
  assets to code by a serialized string tag field, not by asset name — to verify a configuration without
  the Editor, resolve the GUID via `grep -rl "guid: <g>" Assets | grep .meta` and read the asset's
  tag/parameters directly; faster and more reliable than opening the Inspector.

- Author many similar ScriptableObject/data assets by generating YAML, not by hand: when a task needs
  dozens of near-identical `.asset` variants, hand-editing that much YAML is error-prone and Editor-free
  anyway. A short generator script (one call per asset, a fresh GUID for each `.meta`, a printed
  name→GUID manifest) produces all of them plus any list-patch in one deterministic pass, and re-tuning a
  value later is a one-line edit + rerun. Two things that make it safe: renaming an existing asset keeps
  its GUID, so `git mv` + editing the name/tag field re-purposes a legacy asset into a new naming scheme
  with zero prefab/scene churn (prefer that over delete+recreate); serialized *lists* on a component are
  plain YAML sequences — insert entries right before the next scalar key of the same block, locating the
  block by index rather than by regex when the same key name appears on several components in the file.
  Generic rule for tween/text-effect packages with a "one-shot" mode: the effect's contribution vanishes
  the frame it completes, so every *entrance* animation must end at identity (scale 1, offset 0, angle 0)
  or the element visibly snaps; exits are exempt only if the code hides the object before the effect
  completes.

- DOTween in asmdef projects: the Modules half is invisible. Dropping the Asset Store DOTween into a
  Plugins folder gives asmdef assemblies less API than expected — the core DLL is auto-referenced (no
  asmdef reference entry or explicit init needed; confirm with a headless `dotnet build`, not by
  assuming), but the Modules (UI/TMP/Sprite/Physics shortcuts) live with no asmdef and compile into the
  default assembly, which an asmdef assembly cannot reference. Net effect: `Transform` shortcuts work,
  but `TMP_Text`/`Image`/`RectTransform` shortcut extensions do not — reach for the generic
  `DOTween.To(() => x.value, v => x.value = v, target, dur)` instead of adding an asmdef for the plugin.
  Game-feel notes worth reusing when a designer says an effect reads as too calm: the escalation that
  actually reads is a scale pop + a punch-position + a punch-rotation joined at t=0 (a scale-only tween
  never reads as impact regardless of curve); tune the *visible peak*, not the tween target, since
  overshoot eases land meaningfully past their nominal target and can shove a label off-screen; "different
  every time" is cheaper and stronger as continuous `Random.Range` inside a per-tier band than a pool of
  authored presets; and when replacing a per-frame timer state machine with one owned `Sequence`, cache
  the authored starting transform in `Awake` (a killed sequence leaves the transform anywhere), `Kill()`
  the field before rebuilding, and remember `Kill()` defaults to not completing so `OnComplete` won't
  fire — sequences also can't contain infinite loops, so a "hold with idle wobble" is a finite yoyo whose
  period is `hold / loops`.

- Verify third-party DLL semantics from IL, don't guess: when a task hinges on whether a closed-source
  plugin API misbehaves with certain arguments, disassemble instead of reasoning from docs (e.g.
  `monodis` against the plugin DLL, run with `MONO_PATH` pointed at a Unity-bundled Mono runtime that
  actually has `UnityEngine.dll` available — the Editor's own managed folder may not, in which case a
  PlaybackEngine variation does). A few minutes of IL reading turns a "should be fine" into a provable
  fact about what the method actually does with those inputs.

- When the Editor is unreachable, clone an authored object instead of writing prefab YAML by hand for a
  brand-new object: hand-writing a new GameObject into a `.prefab` blind (renderer + material + font/asset
  GUIDs + sorting) is the kind of change whose mistakes only surface when someone opens the Editor.
  `Instantiate(existingObject, transform)` in `Awake`, then overriding just the properties that differ,
  gives a fully-configured clone with zero new authoring surface and zero missing-reference risk, and it
  stays correct if the original is retargeted later. Clone in `Awake` (before anything has moved/tinted
  the source) and reset scale/rotation from cached authored values. Related sizing lesson: for a
  full-screen element that must start and end fully off-screen, don't use a fixed viewport offset
  constant — the element is often wider than the screen; take the viewport 0/1 world edges and pad by
  the element's own measured size, so it's independent of content length, font scale, and aspect ratio.

- Verify a merge resolution by diffing against *both* parents, not just "no conflict markers + it
  compiles." `git diff <parentA> <mergeCommit> -- <paths>` and the same against `<parentB>` — each diff
  should contain *only* the other parent's intended changes; an empty diff against a parent for files you
  didn't touch is proof that side survived verbatim. For a file where both sides added scattered call
  sites, compare call-site inventories (grep + sort + uniq -c) rather than the diff — equal multisets
  means nothing dropped, in one glance. Two traps: Unity's Editor log is append-only and full of stale
  errors — check the *line number* of the last error against the file's total line count, not just the
  last match, before trusting it as current; and conflict-marker greps must force text mode and cover
  untracked files (plain `grep -I` skips binary-flagged YAML/asset files you most want to check) — use a
  `find | xargs grep` pipeline pruning `Library/ Temp/ Logs/ obj/ .git/`. Also: a merge that brings in a
  new asset folder can arrive missing that folder's own `.meta` file even though its contents came
  through — it shows up as a lone untracked entry in git status and is easy to dismiss as editor noise;
  commit it, or every machine generates a different folder GUID.
