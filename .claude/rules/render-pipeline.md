---
paths:
  - "Assets/**/*.mat"
  - "Assets/**/*.shadergraph"
  - "Assets/**/*.vfx"
  - "Assets/_Project/Scenes/*.unity"
  - "Assets/Universal*.asset"
  - "Assets/_Project/Core/**"
  - "ProjectSettings/GraphicsSettings.asset"
  - "ProjectSettings/QualitySettings.asset"
  - "Packages/manifest.json"
---

# Render pipeline (URP)

The project renders with **URP 17.5** (`Assets/UniversalRenderPipelineAsset.asset`). HDRP was
removed in T-1345; do not reintroduce HDRP assets, shaders or components.

- **There is no HDRP → URP converter.** The Render Pipeline Converter in URP 17.5 covers
  Built-in, PPv2 and 2D only. Conversion is manual: ShaderGraph targets by editing the
  graph JSON, materials by script (property remap below).
- **ShaderGraph:** a graph whose only target is HDTarget does not render in URP. Replace the
  target with `UniversalTarget` and the Lit/Unlit sub-target. HDRP-only nodes have no URP
  equivalent: `HD Scene Color` → `Scene Color`, drop `Exposure` (treat it as 1), drop the
  Diffusion Profile block and property. A graph reading `Scene Color` must be Transparent:
  URP's opaque texture is copied after opaques, so an opaque object reading it gets garbage.
  `Scene Color` also needs **Opaque Texture** on in the URP asset.
- **HDRP/Lit → URP/Lit property map:** `_BaseColorMap`→`_BaseMap`, `_NormalMap`→`_BumpMap`,
  `_MaskMap`→`_MetallicGlossMap` (R metal, A smooth) and `_OcclusionMap` (G),
  `_SurfaceType`→`_Surface`, `_AlphaCutoffEnable`→`_AlphaClip`, `_DoubleSidedEnable`→`_Cull 0`.
  Then call `BaseShaderGUI.SetMaterialKeywords(mat, LitGUI.SetMaterialKeywords)` or the
  keywords stay wrong.
- **HDRP emission and light values are exposure-relative; URP has no exposure.** The HDRP
  scene used a fixed 11.5 EV exposure, so values were authored thousands of times too bright
  for URP:
  - An HDRP/Lit `_EmissiveColor` white at 1 nit was invisible in HDRP and renders solid white
    in URP. Those were set to black.
  - `PlayerShaderGraph` emission (`_ColorEmission` 1024, and the `_OutlineColor * 2^10` that
    `PlayerVFX` animates) goes through an `HDRP Exposure Compensation` multiply (×0.000625)
    in front of the Emission block. Keep that node; scripts and materials still use the
    HDRP-scale values.
  - Lights: sun 7937 lux → 1.6; `PlayerVFX._outlineLightIntensity` 100 → 0.3 (the script
    multiplies it by up to 10); `ImpactVFX._lightIntensity` 1000 → 8. Grep for `intensity`
    fields when a light-driven effect floods the screen.
  - VFX Graph colors (up to ~770) looked fine in URP and were left as authored.
- **VFX Graph is its own package.** `com.unity.visualeffectgraph` arrived as an HDRP
  dependency; removing HDRP without listing it in `Packages/manifest.json` removes every
  VFX Graph in the project.
- **Remove the HDRP package before its assets.** While HDRP is still loaded it recreates
  `Assets/HDRPDefaultResources/` and re-adds its entry to
  `GraphicsSettings.m_RenderPipelineGlobalSettingsMap` on save. Delete those after HDRP is
  gone; drop the map entry through `SerializedObject` in the Editor, since a hand edit of
  `GraphicsSettings.asset` is overwritten from memory.
- **Restart the Editor after removing HDRP.** In the same session URP throws
  `Exception: Blitter is already initialized` on every repaint, because HDRP's Blitter is
  still alive. It is Editor state, not a project error; reopen the project.
- Removing assets through `AssetDatabase.DeleteAsset` from the Unity MCP fails with
  `User interactions are not supported for MCP tool calls`. Move or delete with `git mv` /
  `git rm` and call `AssetDatabase.Refresh()`.
- HUD: the `UI Camera` is an **Overlay** camera in `MainCamera`'s camera stack. A new screen
  overlay camera has to be added to that stack, or it does not render.
