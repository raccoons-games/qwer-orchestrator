---
purpose: cross-project accumulated lessons for the unity-developer role
audience: Claude, any RaccoonsGames project generating or running a unity-developer subagent
---

# unity-developer — accumulated lessons

No accumulated lessons yet — star-master is the first project using this role. Contribute short, timestamped, sourced entries here as you learn things worth carrying forward to the next project (not project-specific trivia — that belongs in the repo's own CLAUDE.md).

- 2026-07-16 [star-master] For Unity WebGL, use `!Input.touchSupported` to detect desktop browser vs mobile browser at runtime. `Application.isMobilePlatform` and `SystemInfo.deviceType` are unreliable in WebGL builds and should not be used for this purpose. Evaluate once in `Awake`/`Start`, store in a static property (e.g. `PlatformUtils.IsDesktopWeb`), and keep it a one-time decision rather than a per-frame check.
