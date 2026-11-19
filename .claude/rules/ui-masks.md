---
paths:
  - "Assets/_Project/Prefabs/UI/**"
  - "Assets/_Project/Scenes/*.unity"
---

# UI masks

- UI masking uses the built-in `UnityEngine.UI.Mask`
  (script GUID `31a19414c41e5ae4aae2af33fee712f6`). coffee SoftMask was removed in
  the Unity 6 port; do not reintroduce it.
- A mask whose script is missing still renders its Image. `Canvas/SkillButtons/Mask` is
  a 412x1736 white Image rotated -45° with `showMaskGraphic: false`; without its
  script it showed as a white diagonal band across the screen. When a white or solid
  shape appears in the Game view, look for a "Missing Script" on a mask object first.
- When swapping a mask component, keep its `showMaskGraphic` value.
- A missing script whose serialized fields are `m_Inverse`, `m_MaskInteraction`,
  `m_UseStencil`, `m_RaycastFilter` is a leftover SoftMaskable (GUID
  `97bc2ebab6563400c95b036136d26ea6`). Children of a built-in `Mask` need no such
  component, so remove it: `PrefabUtility.LoadPrefabContents`,
  `GameObjectUtility.RemoveMonoBehavioursWithMissingScript` on every child,
  `SaveAsPrefabAsset`. Fix it in the base prefab (`ComboIndicator`); its variants
  inherit the removal.
