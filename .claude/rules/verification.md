---
paths:
  - "Assets/_Project/**"
  - "Assets/**/*.cs"
---

# Checking a change in Play

- The Unity MCP server (`unity`) drives the open Editor. `Unity_RunCommand` runs C#
  in it: set `EditorApplication.isPlaying = true` to enter Play and `false` to leave.
  The class must be `internal class CommandScript : IRunCommand`.
- **Check `EditorApplication.isPlaying` before editing scripts.** The owner may leave
  the Editor in Play. Scripts recompiled during Play are hot-reloaded into the
  running scene without re-running `Start`. Expect `Parameter 'Hash 0' does not exist`
  and `NullReferenceException`s from `OnEnable`/`OnDisable`, and `_Diag` boot methods
  that never run. Setting `isPlaying = true` then does nothing. Leave Play
  (`isPlaying = false`), refresh, and enter again, each in its own `Unity_RunCommand`:
  `AssetDatabase.Refresh` in the same call as leaving Play is refused as a "user
  interaction", and so is `File.Delete`. Delete files from the shell.
- **Check for compile errors first.** If scripts do not compile, Play silently does not
  start (`All compiler errors have to be fixed before you can enter playmode!`). Read
  `Unity_GetConsoleLogs` with `logTypes: "error"`, or check
  `EditorUtility.scriptCompilationFailed`, before waiting on Play output.
- A script added from outside the Editor is not compiled until the Editor refreshes;
  call `AssetDatabase.Refresh()` through `Unity_RunCommand`.
- For behaviour over time, put a temporary editor-only script under `Assets/_Diag/`
  (`#if UNITY_EDITOR`, `[RuntimeInitializeOnLoadMethod]`) that writes to
  `Logs/<name>.txt`, then read the file. Delete `Assets/_Diag/` and its `.meta` before
  committing.
- To drive combat from a `_Diag` script, call `InputManager`'s handlers directly
  (`OnAttack`, `OnPhoenixShift`, `OnDodge`, `MoveInput`); ability mode is the private
  `_isInEikonicAbilityMode` field, set by reflection. An exception thrown by a game
  handler propagates into the driver and silently ends its coroutine, so wrap each
  press in `try`/`catch` and record the exception.
- **Count console output in the `_Diag` script, not with `Unity_GetConsoleLogs`.** That
  tool returns at most 200 entries, so one flood (hundreds of identical errors) hides
  every other kind. Hook `Application.logMessageReceived` and key each entry by its
  first line plus the first `_Project/Scripts` stack frame.
- `Unity_RunCommand` rejects `using System.Reflection`; put reflection in the `_Diag`
  script instead.
- To prove a fix, run the same driver once with the fix reverted
  (`git diff -- <paths> > patch`, `git checkout -- <paths>`, Play, `git apply patch`)
  and check the error shows up there and is gone with the fix.
- Known Editor noise, unrelated to gameplay: repeated `TargetParameterCountException`
  from the SerializedCollections property drawer.
