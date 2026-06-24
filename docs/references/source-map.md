# Source Map

Paths into the UdonSharp repository at <https://github.com/vrchat-community/UdonSharp>. All paths are relative to `Packages/com.vrchat.UdonSharp/`.

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
| `UdonSharpRuntimeLogWatcher.cs` | Tails the VRChat client log for in-editor diagnostics. |
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
| `Binder/`, `Emit/`, `Core/`, `Assembly/`, `Udon/` | Compiler phases. |

### `Editor/Editors/`

| File | Contains |
|---|---|
| `UdonSharpGUI.cs` | Inspector-style draw helpers — `DrawDefaultUdonSharpBehaviourHeader`, etc. |
| `UdonSharpUndo.cs` | Undo-tracked `AddComponent` / `DestroyImmediate`. |
| `UdonSharpComponentExtensions.cs` | `AddUdonSharpComponent<T>()` extension. |

### `Editor/Serialization/`

| File | Contains |
|---|---|
| `ProxySerializationPolicy.cs` | The `RootOnly` / `All` / `Default` / `NoSerialization` policy constants. |

## Samples (under `Samples~/`)

These are not compiled by default; copy via Package Manager → Samples to use them.

| Directory | Contains |
|---|---|
| `Tutorials/SpinningCubes/` | The classic getting-started series. |
| `Utilities/` | Drop-in components: `InteractToggle`, `BoneFollower`, `TrackingDataFollower`, `WorldAudioSettings`, synced toggle variants. |
| `CustomInspectors/` | Reference custom-inspector implementations. |

## Tests (under `Tests~/`)

Integration scenes and per-feature smoke tests. Useful when you want to see "is this thing actually supported?" — if it's in a test that runs, it works.

## How this reference cites them

When you see **[Source]** in a doc page, look for the corresponding file above. When you see line numbers (e.g. `Runtime/UdonSharpBehaviour.cs:263`), they refer to the file as of the last commit indexed by this reference — your local copy may differ slightly.

## See also

- [Official Docs](official-docs.md)
- [Community Resources](community-resources.md)
