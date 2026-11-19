# Unity 6 upgrade (branch: unity6-upgrade)

Target editor: **Unity 6000.6.9f1** (installed locally; newer than the 6000.0 LTS line).
Original project: Unity 2021.3.15f1.

## What this branch changes

- `ProjectSettings/ProjectVersion.txt` → `6000.6.9f1 (8c45277fb9c5)`
- `Packages/manifest.json`:
  - `com.unity.render-pipelines.high-definition` / `universal`: 12.1.8 → **17.5.0**
    (17.5.x is the HDRP line for the 6000.6 stream)
  - `com.unity.cinemachine`: 2.9.1 → **3.1.3**
  - `com.unity.inputsystem`: 1.4.4 → **1.14.2**
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

1. Unity Hub → Installs → **6000.6.9f1** (already installed;
   include Windows Build Support module).
2. Projects → Add → select this folder (must contain `Assets/`, `ProjectSettings/`).
3. Open with 6000.6.9f1. First import takes a long time (HDRP shader compilation).
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

## White diagonal band (fixed)

Cause: the `coffee SoftMask` package was removed during migration, but
`Canvas/SkillButtons/Mask` still referenced its script. With the script
missing, the white mask Image (412x1736, rotated -45°,
`ShowMaskGraphic: false`) rendered unmasked as a fullscreen diagonal band.

Fix: replaced all 6 `SoftMask` components with built-in
`UnityEngine.UI.Mask` (`showMaskGraphic` preserved), no new dependencies:
- `Assets/_Project/Scenes/MainScene.unity` — `SkillButtons/Mask`, skill-slot `Mask`
- `Assets/_Project/Scenes/LIghtweightScene.unity` — same two
- `Assets/_Project/Prefabs/UI/ComboIndicator.prefab` — `Mask`, `ActionNameMask`

## Player falls through the terrain on Play (fixed)

Cause: all `com.unity.modules.*` entries were dropped from
`Packages/manifest.json` to get past the missing `com.unity.modules.vr@1.0.0`.
Built-in modules are not implicit: without `com.unity.modules.terrainphysics`
the `TerrainCollider` type does not exist, so the terrain had no collider and
the player (and any Rigidbody) fell forever.

Fix: restored every module the original manifest listed except
`com.unity.modules.vr`, which 6000.6 no longer ships.

## Rollback

`git checkout main` returns to the Unity 2021.3.15f1 version.
