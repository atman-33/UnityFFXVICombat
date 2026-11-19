---
paths:
  - "Packages/**"
  - "Assets/csc.rsp"
---

# Packages and built-in modules

- Built-in modules are **not implicit**. Each `com.unity.modules.*` entry in
  `Packages/manifest.json` enables a part of the engine; drop one and its types
  disappear. Dropping all of them removed `com.unity.modules.terrainphysics`, so
  `TerrainCollider` did not exist and the player fell through the terrain on Play.
  When a module fails to resolve, remove only that one (here: `com.unity.modules.vr`,
  not shipped by 6000.6). The Editor's own list is
  `<Editor>/Data/Resources/PackageManager/BuiltInPackages/`.
- A missing module shows as `CS1069 ... forwarded to assembly
  'UnityEngine.<Name>Module' ... Enable the built in package`, but only in code that
  names the type. Scenes that merely use the component fail silently.
- Upgrade a package in steps when its API changed across versions (Input System went
  1.14.2 → 1.16.0 → 1.17.0 → 1.18.0 → 1.19.0), compiling after each.
- Before removing a package, search scenes and prefabs for its script GUIDs, not just
  `using` lines. Removing coffee SoftMask left `Canvas/SkillButtons/Mask` without its
  script (see `ui-masks.md`).
- `Assets/csc.rsp` (`-nowarn:0619`) is a temporary backstop for obsolete-as-error APIs
  in embedded third-party code. Do not add more suppressions to it; fix the code.
