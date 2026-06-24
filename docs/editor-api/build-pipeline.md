# Build Pipeline

UdonSharp integrates with the VRChat SDK build pipeline at two points. Both implementations are internal to the package but reading them clarifies what runs when.

Source: `Editor/BuildUtilities/`. **[Source]**

## `UdonSharpBuildCompile`

Implements `IVRCSDKBuildRequestedCallback`. Runs **before** a world build. Triggers a full U# compile pass to make sure the program assets being uploaded match the latest source.

```csharp
public class UdonSharpBuildCompile : IVRCSDKBuildRequestedCallback {
    public int callbackOrder => /* low */;
    public bool OnBuildRequested(VRCSDKRequestedBuildType requestedBuildType) {
        UdonSharpProgramAsset.CompileAllCsPrograms(forceCompile: false, editorBuild: true);
        return true;
    }
}
```

(Simplified.) Authors typically don't interact with this directly.

## `UdonSharpBuildChecks`

Also `IVRCSDKBuildRequestedCallback`. Runs after the compile pass and aborts the build if any program asset has an error.

If you've been editing scripts when you click Build, this is what saves you from uploading a world that won't compile.

## Adding your own pre-build hook

There is currently no `[PublicAPI]` extension hook on UdonSharp's pipeline. If you need to inject behaviour before U# compiles, just add your own `IVRCSDKBuildRequestedCallback`:

```csharp
#if UNITY_EDITOR
using VRC.SDKBase.Editor.BuildPipeline;

public class MyPreBuild : IVRCSDKBuildRequestedCallback {
    public int callbackOrder => -100;   // run before U# compile

    public bool OnBuildRequested(VRCSDKRequestedBuildType type) {
        // Validate, sanitize, set version numbers, etc.
        if (somethingWrong) {
            EditorUtility.DisplayDialog("Build blocked", "...", "OK");
            return false;
        }
        return true;
    }
}
#endif
```

If your callback's `callbackOrder` is higher than U#'s (typical default), you run after the compile. Use `UdonSharpProgramAsset.AnyUdonSharpScriptHasError()` to bail out if the compile failed.

## Order of operations during build

1. VRChat SDK collects build callbacks.
2. They run in `callbackOrder` (lowest to highest).
3. `UdonSharpBuildCompile` triggers compile.
4. `UdonSharpBuildChecks` verifies no errors.
5. SDK proceeds to scene processing.
6. The world is packed and uploaded.

## Debugging in-game (community-known)

VRChat ships in-game debug views accessible via **`RShift + ` + N`** (where N selects the page):

| Page | Content |
|---|---|
| 1 | AssetBundle / memory info |
| 2 | Version & info |
| 3 | Log viewer — `Debug.Log`, UdonBehaviour crashes, Unity errors |
| 4 | Players (incl. "Suffering" field flagging excessive per-player Udon data) |
| 6 | Net Objects — ownership, group, bandwidth, sync interval |
| 7–0 | Overlays: PhysBones, network objects, players, UI shapes |

World creator gate applies to view 6, Audio Sources, and overlays unless "World Debugging" is enabled. **[Docs]** <https://creators.vrchat.com/worlds/udon/world-debug-views/>

VRChat launch flags for verbose Udon logging: `--enable-udon-debug-logging` and `--enable-debug-gui`. Set via VRC Quick Launcher in VCC, a batch file, or Steam launch options. **[Docs]** <https://creators.vrchat.com/worlds/udon/debugging-udon-projects/>

Log file location (PC): `C:\Users\<user>\AppData\LocalLow\VRChat\VRChat\output_log_<timestamp>.txt`. Contains UdonBehaviour exceptions with script + line, all `Debug.Log` output, asset-bundle load events, network warnings. **[Docs]** <https://help.vrchat.com/hc/en-us/articles/9521522810899-Where-do-I-find-my-Output-Logs-and-Crash-Dumps>

## Build-validator community tools

- **VRWorld Toolkit** (oneVR) — 90+ scene-validator checks (lighting, post-processing, SDK config, optimization), pre-build Mass Texture Importer, the `DisableOnBuild` GameObject tag. **[Community]** <https://github.com/oneVR/VRWorldToolkit>
- **EasyQuestSwitch** (Jordo, VRChat-curated) — pre-build pipeline component swapping for PC ↔ Android cross-platform builds. **[Community]**

## See also

- [Program Asset](program-asset.md) — methods you might want from a build hook.
- [Editor Utility](editor-utility.md)
- [References → Community Resources](../references/community-resources.md)
