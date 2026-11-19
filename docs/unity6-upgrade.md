# Unity 6 upgrade

`main` runs the upstream Unity 2021.3.15f1 sample on **Unity 6000.6.9f1** with
**URP**. The upstream 2021.3 HDRP project is the tag `upstream-2021.3`;
`git checkout upstream-2021.3` rolls back.

This page is the procedure and the current state. The step-by-step history of
the migration (each error and its fix) lives in the vault:
`C:/repos/workhub-vault/projects/0220-UnityFFXVICombat/backlog/B-001-unity6-migration/010-error-fix-log.md`
(Unity 6) and `.../backlog/B-002-hdrp-to-urp/` (HDRP → URP).

## Open the project

1. Unity Hub → Installs → **6000.6.9f1** (with Windows Build Support).
2. Projects → Add → this folder.
3. Open it. Accept the Input System restart prompt if it appears.
4. Play `Assets/_Project/Scenes/MainScene.unity`.
   Controls: WASD move, LMB attack, RMB Phoenix Shift, Space dodge,
   Shift abilities.

## Package state

`Packages/manifest.json` against upstream:

- Upgraded: URP 17.5.0, Cinemachine 3.1.3, Input System 1.19.0,
  Timeline 1.8.7, TextMeshPro 3.0.9, Test Framework 1.4.5, uGUI 2.0.0,
  coffee ui-particle 4.14.2.
- Added: Visual Effect Graph 17.5.0 (was an HDRP dependency).
- Removed: HDRP, `com.unity.modules.vr` (not shipped by 6000.6), Terrain Tools,
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

## HDRP → URP

The render pipeline is `Assets/UniversalRenderPipelineAsset.asset` for every
quality level (HDR, depth and opaque textures on). The look differs from the
HDRP original; matching it was not a goal. `.claude/rules/render-pipeline.md`
has the mapping and the traps.

- ShaderGraphs (Player, Projection, Distortion, Blood, Rock, TerrainGrass,
  SpeedTree Bark / Branch) target URP Lit / Unlit. HDRP-only nodes were
  replaced (`HD Scene Color` → `Scene Color`) or dropped (Exposure, Diffusion
  Profile).
- Every HDRP/Lit, HDRP/Unlit and HDRP/TerrainLit material, and the two
  built-in `Particles/Standard Unlit` materials, use the URP equivalents.
- Scenes: HDRP camera / light / probe data, StaticLightingSky, the HDRP
  Compositor and its missing-script `SceneIDMap` objects are gone. `UI Camera`
  is an Overlay camera stacked on `MainCamera`. Sky is
  `Assets/_Project/Core/Skybox_Procedural.mat`, fog is linear scene fog, post
  processing is `Assets/_Project/Core/URP Volume Profile.asset`.
- Light intensities were rescaled from HDRP's physical units (sun 7937 → 1.6,
  player outline light 100 → 0.3, impact light 1000 → 8). The player
  ShaderGraph scales its HDRP-scale emission by 0.000625.
- Removed HDRP-only assets: `Assets/Compositor/`, `Assets/HDRPDefaultResources/`,
  `Assets/Samples/High Definition RP/`, the HDRP pipeline asset, volume and
  diffusion profiles, `HDRPShaderGraphs/Lit_Graph`, TMP's HDRP graphs,
  `ProjectSettings/HDRPProjectSettings.asset`.
