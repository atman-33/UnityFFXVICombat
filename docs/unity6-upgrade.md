# Unity 6 upgrade

Branch `unity6-upgrade` runs the upstream Unity 2021.3.15f1 sample on
**Unity 6000.6.9f1**. `main` is still the upstream 2021.3 project;
`git checkout main` rolls back.

This page is the procedure and the current state. The step-by-step history of
the migration (each error and its fix) lives in the vault:
`C:/repos/workhub-vault/projects/0220-UnityFFXVICombat/backlog/B-001-unity6-migration/010-error-fix-log.md`.

## Open the project

1. Unity Hub → Installs → **6000.6.9f1** (with Windows Build Support).
2. Projects → Add → this folder.
3. Open it. The first import is long (HDRP shader compilation). Accept the
   Input System restart prompt if it appears.
4. Play `Assets/_Project/Scenes/MainScene.unity`.
   Controls: WASD move, LMB attack, RMB Phoenix Shift, Space dodge,
   Shift abilities.

## Package state

`Packages/manifest.json` against upstream:

- Upgraded: HDRP / URP 17.5.0, Cinemachine 3.1.3, Input System 1.19.0,
  Timeline 1.8.7, TextMeshPro 3.0.9, Test Framework 1.4.5, uGUI 2.0.0,
  coffee ui-particle 4.14.2.
- Removed: `com.unity.modules.vr` (not shipped by 6000.6), Terrain Tools,
  `com.alelievr.hdrp-ui-camera-stacking`, coffee SoftMask, postprocessing,
  visualscripting, collab-proxy, feature.2d, feature.development.
- Every other built-in `com.unity.modules.*` entry is kept; see
  `.claude/rules/packages.md` for why.
- `Assets/csc.rsp` holds `-nowarn:0619` as a temporary backstop for
  obsolete-as-error APIs in embedded third-party code.

## Code and asset changes

- `Cinemachine` → `Unity.Cinemachine` namespace.
- `Assets/TextMesh Pro/Examples & Extras` deleted (incompatible, unused).
- All 6 `SoftMask` components replaced with built-in `UnityEngine.UI.Mask`
  (`MainScene`, `LIghtweightScene`, `ComboIndicator.prefab`).
- `PlayerMovement.Gravity()` terminal-velocity check fixed
  (`> -_terminalVelocity`).
- HDRP Compositor objects in the scenes are inert since the camera-stacking
  package was removed.

## Remaining manual checks

- Full combat loop in Play (combo, Phoenix Shift, dodge / counter, abilities).
- Pink materials → reimport the HDRP ShaderGraphs.
- UI overlay without the HDRP Compositor (native camera stacking if needed).
