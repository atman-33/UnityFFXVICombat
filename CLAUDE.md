# UnityFFXVICombat

Fork of gabriel0223/UnityFFXVICombat (FFXVI-style third-person combat), ported
from Unity 2021.3 / HDRP to **Unity 6000.6.9f1 / URP**. Kept as a playable reference
for action-RPG combat work. `README.md` is upstream's description; do not edit it.

## Where things are

- `Assets/_Project/Scripts/` — game code, one folder per family (`AI/`, `Combat/`,
  `Movement/`, `Core/`, `States/`, ...). `Core/InputManager.cs` holds live input;
  `Movement/PlayerMovement.cs` the CharacterController movement.
- `Assets/_Project/Scenes/MainScene.unity` — the scene to play. `LIghtweightScene.unity`
  is the lighter variant.
- `Assets/Plugins/` (DOTween, SerializedCollections, UnitySweeper) and
  `Assets/UnityTechnologies/` are embedded third-party code; leave them as shipped.
- `docs/` — human guides; `docs/README.md` lists them. `docs/unity6-upgrade.md` is the
  current package state and how to open the project.
- `.claude/rules/` — development rules; each file loads when a matching path is touched.
- Project notes live in the vault: start at
  `C:/repos/workhub-vault/projects/0220-UnityFFXVICombat/README.md`, then the
  `backlog/B-NNN-*/` item for the work. If that path cannot be read, work from the
  repository alone.

## House rules

- `main` is the Unity 6 / URP project. Branch from it as `feature/<task-id>-<slug>`.
  Upstream's original 2021.3 HDRP state is the tag `upstream-2021.3`; never push to
  the `upstream` remote. Conventional Commits in English; code, comments and
  docs are English.
- The owner's working tree often carries Editor-side changes (`.mat`, `.unity`,
  `ProjectSettings/`). Commit only the files your change touched.
- A development rule discovered while working goes into `.claude/rules/`: one topic per
  file, `paths:` frontmatter naming the files it governs. `CLAUDE.md` stays short and
  holds only what applies to every session.
- Done means the project compiles with no errors and the change is checked in Play
  (`.claude/rules/verification.md`). Feel (camera, combat timing) is the owner's call:
  report it as "built, not played" when you could not judge it.
- When a check cost you time, write the lesson into `.claude/rules/` in the same change.
