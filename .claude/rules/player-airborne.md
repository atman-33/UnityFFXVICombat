---
paths:
  - "Assets/_Project/Scripts/Movement/PlayerMovement.cs"
  - "Assets/_Project/Scripts/States/**/*.cs"
  - "Assets/_Project/Scripts/Combat/**/*.cs"
  - "Assets/_Project/Animations/Player/PlayerAC.controller"
---

# Player jump and airborne states

- Only `PlayerIdleMovementState` calls `PlayerMovement.Move()`, which moves the player
  horizontally and vertically in one `CharacterController.Move`. Every other state
  (attack, dodge, Phoenix Shift) used to leave vertical velocity unapplied, so a
  player who entered one mid-air hung there. `PlayerMovement.LateUpdate` now applies
  the vertical part on its own when `Move()` did not run that frame.
- Do not add a second, vertical-only `Move` in the same frame as `Move()`:
  `CharacterController.velocity` reflects the last `Move` call, and `Move()` reads it
  as the current horizontal speed, so walking would restart from zero every frame.
- Vertical movement is skipped while `Animator.applyRootMotion` is on; Eikonic
  abilities drive the player with root motion (`EikonicAbility.Activate`).
- Abilities are refused mid-air (`AbilityManager.HandleAbilityPressed` checks
  `PlayerMovement.Grounded`): each places its timeline and VFX at the player's
  position on activation.
- `PlayerAC` reaches `SwordSlash1` only on the `Slash1` trigger from a state that has
  that transition. A state without one (as `JumpStart` / `InAir` / `JumpLand` had)
  swallows the trigger, the combo's animation events never fire, and
  `PlayerAttackState` never ends. Any new locomotion state needs a `Slash1` transition
  to `SwordSlash1`. Dodge, Phoenix Shift and TakeDamage come from Any State.
- **An attack pressed mid-air must interrupt the jump transitions.** Otherwise
  `Slash1` waits out `JumpStart -> InAir` and the slash starts near landing. The jump
  states' own transitions use `interruptionSource: Source` with **ordered
  interruption on**, and each state's `-> SwordSlash1` transition is first in its list
  (highest priority), then `-> JumpStart`. Two traps:
  - Ordered interruption **off** lets a transition interrupt itself while its
    condition holds, restarting every frame: the animator freezes mid-transition
    (seen as `InAir -> JumpLand` never completing).
  - Do not make the `-> SwordSlash1` transitions or the Idle `-> JumpStart` /
    `-> InAir` ones interruptible. A slash interrupted right after it was entered
    exits at once, and `PlayerComboController.OnAttackAnimationEnd` reads an empty
    `GetCurrentAnimatorClipInfo` (now guarded); before the guard it threw and
    `PlayerAttackState` never ended.
- Landing while moving skips the landing clip: `JumpLand -> Idle` on `Speed > 0.1`
  (no exit time, 0.1 s) is listed before the exit-time transition. Playing the
  standing land clip while the controller moves reads as sliding.
- The jump clips are the StarterAssets ones in `Assets/Character/Animations/`.
  `JumpLand` fires an `OnLand` animation event; `PlayerMovement.OnLand` is its empty
  receiver. A clip event with no receiver logs an error each time it fires.
- Keyboard Space is Dodge; keyboard jump is F. `PlayerInputAsset.cs` is generated
  from the `.inputactions` file on import, so edit the `.inputactions` and let the
  Editor regenerate the wrapper.
