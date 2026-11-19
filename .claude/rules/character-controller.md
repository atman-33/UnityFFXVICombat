---
paths:
  - "Assets/_Project/Scripts/**/*.cs"
---

# CharacterController and abilities

- `EnemyHealth.Die` disables the enemy's `CharacterController` and leaves the
  GameObject and its other components running. Any `Update` that calls
  `CharacterController.Move` must return early when `!controller.enabled`; otherwise
  Unity logs `CharacterController.Move called on inactive controller` every frame for
  every dead enemy. `EnemyMovement` and `DashController` do this; a dash started by
  the killing blow's knockback is the `DashController` case.
- `AbilityManager._abilities` holds only the configured buttons (MainScene: West,
  Infernal Skyfall). Look an ability up with `TryGetValue`; a button with no ability
  does nothing, matching the HUD, which leaves that slot empty
  (`SkillButtonsManager.Start`).
