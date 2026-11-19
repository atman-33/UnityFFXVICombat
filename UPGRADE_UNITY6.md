# Unity 6 upgrade (branch: unity6-upgrade)

Target editor: **Unity 6.0 LTS — 6000.0.64f1** (latest 6000.0 LTS as of Nov 2026).
Original project: Unity 2021.3.15f1.

## What this branch changes

- `ProjectSettings/ProjectVersion.txt` → `6000.0.64f1 (5360b7cd7953)`
- `Packages/manifest.json`:
  - `com.unity.render-pipelines.high-definition` / `universal`: 12.1.8 → **17.0.3**
  - `com.unity.cinemachine`: 2.9.1 → **3.1.3**
  - `com.unity.inputsystem`: 1.4.4 → **1.11.0**
  - `com.unity.timeline`: 1.6.4 → **1.8.7** (used by `InfernalSkyfallTimeline.playable`)
  - `com.unity.textmeshpro`: 3.0.6 → **3.0.9**
  - `com.unity.terrain-tools`: 4.0.3 → **5.0.4**
  - `com.unity.test-framework`: 1.1.33 → **1.4.5**
  - `com.unity.ugui`: 1.0.0 → **2.0.0**
  - Kept: `com.alelievr.hdrp-ui-camera-stacking` 1.4.1, coffee UGUI packages, IDE packages, modules
  - Removed (verified unused in `Assets/`): `postprocessing`, `visualscripting`,
    `collab-proxy`, `feature.2d`, `feature.development`
- Scripts: `Cinemachine` → `Unity.Cinemachine` namespace (`CameraShakeController.cs`);
  removed unused `using Cinemachine;` from `PhoenixShift.cs`, `DodgeController.cs`.
  DOTween (`Assets/Plugins/Demigiant`) and Starter Assets (`Assets/UnityTechnologies`)
  are embedded and version-independent.

## How to open

1. Unity Hub → Installs → install **Unity 6.0 LTS 6000.0.64f1**
   (include Windows Build Support module).
2. Projects → Add → select this folder (must contain `Assets/`, `ProjectSettings/`).
3. Open with 6000.0.64f1. First import takes a long time (HDRP shader compilation).
   Accept the Input System restart prompt if shown.

## Known manual steps (need the Editor)

1. **Cinemachine 2 → 3 scene migration.** `MainScene.unity` / `LIghtweightScene.unity`
   contain Cinemachine 2 virtual cameras. Use the Cinemachine upgrade tool
   (`Cinemachine → Upgrade to Cinemachine 3` if offered) or recreate the vcams;
   then verify follow targets (`CinemachineCameraTarget`) and impulse listeners.
2. **HDRP Compositor** (`com.alelievr.hdrp-ui-camera-stacking` 1.4.1) hooks HDRP
   internals and may not compile against HDRP 17. If it errors, remove the package
   and the `HDRP Compositor` objects, then use HDRP 17 camera stacking instead.
3. Reimport shaders if materials look pink (HDRP ShaderGraphs recompile on first open).
4. Play-test: `Assets/_Project/Scenes/MainScene.unity` → Press Play.
   Controls: WASD move, LMB attack, RMB Phoenix Shift, Space dodge, Shift abilities.

## Rollback

`git checkout main` returns to the Unity 2021.3.15f1 version.
