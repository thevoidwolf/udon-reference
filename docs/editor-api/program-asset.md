# UdonSharpProgramAsset

`UdonSharpEditor.UdonSharpProgramAsset` is a `ScriptableObject` asset that binds a `.cs` source file to its compiled Udon Program. Created automatically when you right-click → Create → U# Script. Source: `Editor/UdonSharpProgramAsset.cs` (bundled at `Packages/com.vrchat.worlds/Integrations/UdonSharp/`). **[Source]**

## Important fields and properties

```csharp
public MonoScript sourceCsScript;                       // the .cs file
public UdonSharpProgramVersion ScriptVersion { get; set; }
public UdonSharpProgramVersion CompiledVersion { get; set; }
```

`ScriptVersion` and `CompiledVersion` differ when the source has been changed but not yet recompiled, or when a project migration introduces a new version of the script format. The editor watches for this divergence.

## Static methods

```csharp
public static UdonSharpProgramAsset[] GetAllUdonSharpPrograms();
public static void CompileAllCsPrograms(bool forceCompile = false, bool editorBuild = true);
public static bool IsAnyProgramAssetSourceDirty();
public static bool AnyUdonSharpScriptHasError();
public static UdonSharpProgramAsset GetProgramAssetForClass(Type classType);
public static Type GetBehaviourClass(UdonBehaviour behaviour);

// Lives on UdonSharpEditorUtility, not on UdonSharpProgramAsset:
UdonSharpEditorUtility.DeleteProgramAsset(UdonSharpProgramAsset programAsset);
```

| Method | Use |
|---|---|
| `GetAllUdonSharpPrograms` | Enumerate every program asset in the project. Caches and auto-repairs stale references. |
| `CompileAllCsPrograms(forceCompile)` | Trigger a recompile of every U# script. `forceCompile=true` skips dirty-detection. |
| `IsAnyProgramAssetSourceDirty` | True if any `.cs` source file has changed since last compile. |
| `AnyUdonSharpScriptHasError` | True if any program has a compile error. **Check before triggering an SDK build.** |
| `GetProgramAssetForClass(typeof(MyScript))` | Find the asset associated with a specific class. |
| `GetBehaviourClass(udonBehaviour)` | What C# class is this UdonBehaviour running? |
| `UdonSharpEditorUtility.DeleteProgramAsset` | Safe deletion (also deletes the linked serialized Udon program asset). **[Source]** |

Also on the menu: `VRChat SDK → Udon Sharp → Refresh All UdonSharp Assets` calls the (public, non-`[PublicAPI]`) `UdonSharpProgramAsset.UdonSharpCheckAbsent()` to rebuild the program-asset cache. **[Source]**

## Instance methods

```csharp
public Type GetClass();
```

Returns the C# class that this program asset compiles.

## Patterns

### Pre-build sanity check

```csharp
#if UNITY_EDITOR
public class PreBuildValidator : IVRCSDKBuildRequestedCallback {
    // U#'s own upload compile runs at callbackOrder 100 (and already aborts on errors).
    // Use > 100 to see this build's results; at 0 you'd be reading the previous compile.
    public int callbackOrder => 101;
    public bool OnBuildRequested(VRCSDKRequestedBuildType type) {
        if (UdonSharpProgramAsset.AnyUdonSharpScriptHasError()) {
            EditorUtility.DisplayDialog("Build blocked", "Fix U# errors first.", "OK");
            return false;
        }
        return true;
    }
}
#endif
```

### Recompile and verify

```csharp
public static bool TryRecompile() {
    UdonSharpProgramAsset.CompileAllCsPrograms(forceCompile: true);
    return !UdonSharpProgramAsset.AnyUdonSharpScriptHasError();
}
```

### Find all places using a specific script

```csharp
var asset = UdonSharpProgramAsset.GetProgramAssetForClass(typeof(MyScript));
foreach (var ub in Object.FindObjectsOfType<UdonBehaviour>()) {
    if (ub.programSource == asset) Debug.Log($"in use on {ub.name}", ub);
}
```

## See also

- [Editor Utility](editor-utility.md)
- [Build Pipeline](build-pipeline.md)
