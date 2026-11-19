---
paths:
  - "Assets/_Project/**"
  - "Assets/**/*.cs"
---

# Checking a change in Play

- The Unity MCP server (`unity`) drives the open Editor. `Unity_RunCommand` runs C#
  in it: set `EditorApplication.isPlaying = true` to enter Play and `false` to leave.
  The class must be `internal class CommandScript : IRunCommand`.
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
- Known Editor noise, unrelated to gameplay: repeated `TargetParameterCountException`
  from the SerializedCollections property drawer.
