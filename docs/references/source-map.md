# Source Map

Where to read the UdonSharp runtime and editor source.

## Where the code lives (as of SDK 3.10.5, Oct 2026)

Since **Worlds SDK 3.4.0** (Oct 4, 2023) UdonSharp — and ClientSim — ship *inside* the Worlds SDK package `com.vrchat.worlds`; the separate `com.vrchat.udonsharp` VPM package is legacy (the Worlds package lists it under `legacyPackages`, so the VCC removes it on upgrade). **[Docs]** <https://creators.vrchat.com/releases/release-3-4-0> · <https://creators.vrchat.com/worlds/udon/udonsharp/>

| Location | Status | Root for the paths below |
|---|---|---|
| **Worlds SDK package** (what your project actually compiles) | Current — tracks every SDK release | `Packages/com.vrchat.worlds/Integrations/UdonSharp/` |
| **GitHub repo** <https://github.com/vrchat-community/UdonSharp> | Not archived, but the package code is frozen at **U# 1.1.9** (last package commit Aug 1, 2023). Later commits (to Oct 2026) are docs-only. | `Packages/com.vrchat.UdonSharp/` |
| Legacy `Assets/UdonSharp/` (0.x `.unitypackage`) | Deprecated; removed by the VCC via `legacyFolders` | — |

ClientSim likewise lives at `Packages/com.vrchat.worlds/Integrations/ClientSim/`; its repo is <https://github.com/vrchat-community/ClientSim>.

All paths below are relative to `Packages/com.vrchat.worlds/Integrations/UdonSharp/`. Every listed file was confirmed present in both the 3.10.5 Worlds package and the GitHub repo on 2026-10-22. Where the two differ, prefer the SDK copy — it contains post-1.1.9 fixes (e.g. 3.10.2's log-parse tool, 3.10.4's `IUdonEventReceiver` on `UdonSharpBehaviour`) that the GitHub repo does not.

## Runtime — `Runtime/`

| File | Contains |
|---|---|
| `UdonSharpBehaviour.cs` | The abstract base class, every lifecycle event stub, type-ID helpers. |
| `UdonSharpAttributes.cs` | `[UdonSynced]`, `[UdonBehaviourSyncMode]`, `[RecursiveMethod]`, `[FieldChangeCallback]`, the `UdonSyncMode` and `BehaviourSyncMode` enums. |
| `UdonSharpRuntimeUtils.cs` | Internal `UdonSharpInternalUtility.GetTypeID(Type)` — the SHA-256-truncated type hash. |
| `UdonSharpLocator.cs` | Internal asset-locator used by the data-asset bootstrap. |
| `UdonSharpDataLocator.cs` | Internal locator for the U# data directory. |
| `Libraries/CompilerInternal/InstantiationShim.cs` | The supported `Instantiate(GameObject, ...)` overloads, aliased to the VRChat extern. |
| `Libraries/CompilerInternal/GetComponentShim.cs` | The compiler shims for `GetComponent` family methods over UdonBehaviour-derived types. |
| `Libraries/CompilerInternal/UdonSharpBehaviourMethods.cs` | Method-stub metadata. |

## Editor — `Editor/`

| File | Contains |
|---|---|
| `UdonSharpEditorUtility.cs` | The main editor-side API: proxy/backing lookups, copy methods, program-asset queries. |
| `UdonSharpEditorManager.cs` | Large coordinator for edit-time U# behaviour. Not usually needed by consumers. |
| `UdonSharpEditorCache.cs` | Compiler caching. |
| `UdonSharpProgramAsset.cs` | The program-asset `ScriptableObject` type + static program-asset helpers. |
| `UdonSharpAssemblyDefinition.cs` | The `UdonSharpAssemblyDefinition` marker `ScriptableObject`. |
| `UdonSharpAssetCompileWatcher.cs` | File-system watcher driving auto-recompile. |
| `UdonSharpUpgrader.cs` | Migration code for old U# projects. |
| `UdonSharpRuntimeLogWatcher.cs` | Tails the VRChat client log for in-editor diagnostics (updated for the new client log format in SDK 3.7.6). |
| `UdonSharpUtils.cs` | Misc editor utilities. |

### `Editor/BuildUtilities/`

| File | Contains |
|---|---|
| `UdonSharpBuildCompile.cs` | `IVRCSDKBuildRequestedCallback` that triggers compile during build. |
| `UdonSharpBuildChecks.cs` | `IVRCSDKBuildRequestedCallback` that blocks the build if any program has errors. |

### `Editor/Compiler/`

The U# compiler itself. Most consumers never need to read this.

| File | Contains |
|---|---|
| `UdonSharpCompilerV1.cs` | Top-level compile driver. |
| `CompilationContext.cs` | Per-compile state. |
| `AbstractPhaseContext.cs` | Compile-phase orchestration. |
| `UdonSharpClass.cs`, `UdonSharpClassDebugInfo.cs` | Class metadata. |
| `UdonSharpHeapFactory.cs`, `UdonSharpLabelTable.cs` | Heap and label-table helpers. |
| `Binder/`, `Emit/`, `Core/`, `Assembly/`, `Udon/` | Compiler phases. The SDK copy adds `Binder/Symbols/ExternSynthesizedFieldSymbol.cs` (not in the GitHub repo). |

### `Editor/Editors/`

| File | Contains |
|---|---|
| `UdonSharpGUI.cs` | Inspector-style draw helpers — `DrawDefaultUdonSharpBehaviourHeader`, etc. |
| `UdonSharpUndo.cs` | Undo-tracked `AddComponent` / `DestroyImmediate`. |
| `UdonSharpComponentExtensions.cs` | `AddUdonSharpComponent<T>()` extension. |
| `LogFileParseUtility.cs` | *SDK-only (not in the GitHub repo).* Backs the "Parse Logs from File" tool (VRChat SDK → Udon Sharp menu) added in SDK 3.10.2. |

### `Editor/Serialization/`

| File | Contains |
|---|---|
| `ProxySerializationPolicy.cs` | The `RootOnly` / `All` / `Default` / `NoSerialization` policy constants. |

## Samples (under `Samples~/`)

These are not compiled by default. The Worlds package registers two of them as Package Manager samples ("UdonSharp Tutorials", "UdonSharp CustomInspectors"); `Utilities/` is not registered, so copy those scripts out of the package folder by hand.

| Directory | Contains |
|---|---|
| `Tutorials/SpinningCubes/` | The classic getting-started series. |
| `Utilities/` | Drop-in components: `InteractToggle`, `BoneFollower`, `TrackingDataFollower`, `WorldAudioSettings`, `PlayerModSetter`, plus synced toggle variants under `Synced/`. |
| `CustomInspectors/` | Reference custom-inspector implementations. |

## Tests (under `Tests~/`)

Integration scenes and per-feature smoke tests. Useful when you want to see "is this thing actually supported?" — if it's in a test that runs, it works.

## How this reference cites them

When you see **[Source]** in a doc page, look for the corresponding file above. When you see line numbers (e.g. `Runtime/UdonSharpBehaviour.cs:263`), they refer to the file as of the last commit indexed by this reference — your local copy may differ slightly, and the SDK-bundled copy (`Packages/com.vrchat.worlds/Integrations/UdonSharp/`) can differ from the frozen GitHub repo.

## See also

- [Official Docs](official-docs.md)
- [Community Resources](community-resources.md)
